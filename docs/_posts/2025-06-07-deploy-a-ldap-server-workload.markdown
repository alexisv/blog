---
layout: post
title:  "Deploy a LDAP server workload"
date:   2025-06-07 13:21:00 -0400
categories: deephackmode.io update
---
I will need a LDAP server as part of the authentication system of my TMC-SM deployment.  This LDAP server is a separate application workload and can be used by other components as well such as Ops Manager, TKGI, etc.  I will be deploying the LDAP server as a kubernetes workload in a TKGI cluster.

### Prepare the TKGI cluster

The only thing I needed to do to prepare the TKGI cluster, is to create a default StorageClass.  That's because there was no default at all, and I need to store the LDAP data in a persistent volume, so that the data will not be wiped out in case of pod restarts.

In the Ops Manager VM, which I use as my jumpbox for my TKGI cluster, I created a YAML file for the StorageClass definition:

{% include codeblock.html code="cat > sc.yaml <<EOF
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: default-sc
  annotations:
      storageclass.kubernetes.io/is-default-class: \"true\"
provisioner: csi.vsphere.vmware.com
allowVolumeExpansion: true
parameters:
  datastoreurl: \"ds:///vmfs/volumes/67f5ab40-e58896f2-b0e5-b49691d2ecdc/\"
  
EOF
" %}

Then, run kubectl apply on the yaml:
{% include codeblock.html code="k apply -f sc.yaml
storageclass.storage.k8s.io/default-sc created
" %}

### Create a namespace for the LDAP app

To keep things organized, I need to create a namespace named "openldap" for the LDAP workload.

{% include codeblock.html code="k create ns openldap
" %}

### Create a certificate and key for LDAP TLS

These are needed so that the LDAP server can use TLS/SSL.  I will create a self-signed certificate. 

Create a SSL config with Subject Alternative Name (SAN).  The config file is named `ldap-cert.conf`.

{% include codeblock.html code="cat > ldap-cert.conf <<EOF
[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
req_extensions     = req_ext
x509_extensions    = v3_ext
prompt             = no

[ req_distinguished_name ]
C  = US
ST = California
L  = San Francisco
O  = Deep Hackmode Inc
OU = Engineering
CN = ldap.deephackmode.io

[ req_ext ]
subjectAltName = @alt_names

[ v3_ext ]
subjectAltName = @alt_names
basicConstraints = CA:FALSE
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth

[ alt_names ]
DNS.1 = ldap.deephackmode.io
EOF
" %}

Generate the Cert and Key:

{% capture ldap_code %}
openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout ldap.deephackmode.io.key \
  -out ldap.deephackmode.io.crt \
  -config ldap-cert.conf
{% endcapture %}

{% include codeblock.html code=ldap_code %}

This gives 2 files:
- `ldap.deephackmode.io.crt` – TLS cert
- `ldap.deephackmode.io.key` – Private key


### Create the TLS secret

This is to store the cert and key in a kubernetes secret that can be used by the workload app (LDAP).

{% capture secret_code %}
kubectl -n openldap create secret tls ldap-tls \
  --cert=ldap.deephackmode.io.crt \
  --key=ldap.deephackmode.io.key
{% endcapture %}

{% include codeblock.html code=secret_code %}


### Create the Deployment definition YAML

Create the definition YAML for the kubernetes deployment, pvc's and service.

{% capture yaml_code %}
cat > openldap.yaml <<EOF
apiVersion: v1
kind: Service
metadata:
  name: openldap-service
spec:
  selector:
    app: openldap
  ports:
    - name: ldap
      port: 389
      targetPort: 389
    - name: ldaps
      port: 636
      targetPort: 636
  type: LoadBalancer

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ldap-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ldap-config
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 512Mi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: openldap
spec:
  replicas: 1
  selector:
    matchLabels:
      app: openldap
  template:
    metadata:
      labels:
        app: openldap
    spec:
      hostname: ldap
      containers:
        - name: openldap
          image: osixia/openldap:latest
          ports:
            - containerPort: 389
            - containerPort: 636
          env:
            - name: LDAP_TLS
              value: "true"
            - name: LDAP_TLS_CRT_FILENAME
              value: "tls.crt"
            - name: LDAP_TLS_KEY_FILENAME
              value: "tls.key"
            - name: LDAP_TLS_CA_CRT_FILENAME
              value: "tls.crt"
            - name: LDAP_TLS_VERIFY_CLIENT
              value: "try"
            - name: LDAP_DOMAIN
              value: "deephackmode.io"
            - name: LDAP_ORGANISATION
              value: "Deep Hack Mode Inc"
            - name: LDAP_TLS_DH_PARAM_FILENAME
              value: "false"
          volumeMounts:
            - name: certs
              mountPath: /container/service/slapd/assets/certs
            - name: certs-target
              mountPath: /certs-readonly
              readOnly: true
            - name: ldap-data
              mountPath: /var/lib/ldap
            - name: ldap-config
              mountPath: /etc/ldap/slapd.d
      initContainers:
        - name: copy-certs
          image: busybox
          command: ["sh", "-c", "cp /certs-readonly/* /certs-writable/"]
          volumeMounts:
            - name: certs
              mountPath: /certs-writable
            - name: certs-target
              mountPath: /certs-readonly
              readOnly: true
      volumes:
        - name: certs
          emptyDir: {}
        - name: certs-target
          secret:
            secretName: ldap-tls
        - name: ldap-data
          persistentVolumeClaim:
            claimName: ldap-data
        - name: ldap-config
          persistentVolumeClaim:
            claimName: ldap-config


EOF
{% endcapture %}

{% include codeblock.html code=yaml_code %}

Create the kubernetes resources from the YAML:
{% include codeblock.html code="k -n openldap apply -f openldap.yaml" %}


Tail the container logs to verify when the server is ready.  

```
k -n openldap logs openldap-778b799856-49nf5 -f
...
***  INFO   | 2025-06-01 18:55:18 | Add image bootstrap ldif...
***  INFO   | 2025-06-01 18:55:18 | Add custom bootstrap ldif...
***  INFO   | 2025-06-01 18:55:18 | Add TLS config...
Generating DH parameters, 2048 bit long safe prime, generator 2
This is going to take a long time
```

It might be stuck in that line for like two minutes.  It should continue eventually.

```
...
***  INFO   | 2025-06-01 18:56:40 | Disable replication config...
***  INFO   | 2025-06-01 18:56:40 | Stop OpenLDAP...
***  INFO   | 2025-06-01 18:56:40 | Configure ldap client TLS configuration...
***  INFO   | 2025-06-01 18:56:40 | Remove config files...
***  INFO   | 2025-06-01 18:56:40 | First start is done...
***  INFO   | 2025-06-01 18:56:40 | Remove file /container/environment/99-default/default.startup.yaml
***  INFO   | 2025-06-01 18:56:40 | Environment files will be proccessed in this order :
Caution: previously defined variables will not be overriden.
/container/environment/99-default/default.yaml

To see how this files are processed and environment variables values,
run this container with '--loglevel debug'
***  INFO   | 2025-06-01 18:56:40 | Running /container/run/process/slapd/run...
683ca268 @(#) $OpenLDAP: slapd 2.4.57+dfsg-1~bpo10+1 (Jan 30 2021 06:59:51) $
        Debian OpenLDAP Maintainers <pkg-openldap-devel@lists.alioth.debian.org>
683ca268 slapd starting

```


In the jumpbox "numbat", which I use to examine my TKGM clusters, install the ldap-utils package to make the `ldapsearch` cli available for use.

{% include codeblock.html code="sudo apt update
sudo apt install ldap-utils
" %}

Run `ldapsearch` using TLS/SSL and the cert file `ldap.deephackmode.io.crt` as the CA file.

```
ubuntu@numbat:~$ env LDAPTLS_CACERT=ldap.deephackmode.io.crt ldapsearch -H 'ldaps://ldap.deephackmode.io' -b dc=deephackmode,dc=io -D "cn=admin,dc=deephackmode,dc=io" -w admin
# extended LDIF
#
# LDAPv3
# base <dc=deephackmode,dc=io> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# deephackmode.io
dn: dc=deephackmode,dc=io
objectClass: top
objectClass: dcObject
objectClass: organization
o: Deep Hackmode Inc
dc: deephackmode

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
ubuntu@numbat:~$
```


Create a LDIF file with example user and group records for our use case.

{% capture ldif_code %}
cat > ldap.ldif <<EOF
# groups, deephackmode.io
dn: ou=groups,dc=deephackmode,dc=io
objectClass: organizationalUnit
ou: groups

# users, deephackmode.io
dn: ou=users,dc=deephackmode,dc=io
objectClass: organizationalUnit
ou: users

# alexisv, users, deephackmode.io
dn: cn=alexisv,ou=users,dc=deephackmode,dc=io
objectClass: simpleSecurityObject
objectClass: inetOrgPerson
objectClass: posixAccount
cn: alexisv
sn: Villalon
mail: alexisv@deephackmode.io
description: alexisv
userPassword:: e1NTSEF9RC9IZ2FDV2t6SG03alVxRVl1SmJ1QVRWNVBBdTV6NXM=
uid: alexisv
uidNumber: 1000
gidNumber: 1000
homeDirectory: /home/alexisv

# testuser, users, deephackmode.io
dn: cn=testuser,ou=users,dc=deephackmode,dc=io
objectClass: simpleSecurityObject
objectClass: inetOrgPerson
objectClass: posixAccount
cn: testuser
sn: User
mail: testuser@deephackmode.io
description: Test User
userPassword:: e1NTSEF9RC9IZ2FDV2t6SG03alVxRVl1SmJ1QVRWNVBBdTV6NXM=
uid: testuser
uidNumber: 1001
gidNumber: 1000
homeDirectory: /home/testuser

# tmc-admins, groups, deephackmode.io
dn: cn=tmc-admins,ou=groups,dc=deephackmode,dc=io
objectClass: groupOfNames
objectClass: top
cn: tmc-admins
description: tmc sm admins
member: cn=alexisv,ou=users,dc=deephackmode,dc=io

# tmc-members, groups, deephackmode.io
dn: cn=tmc-members,ou=groups,dc=deephackmode,dc=io
objectClass: groupOfNames
objectClass: top
cn: tmc-members
description: tmc sm members
member: cn=alexisv,ou=users,dc=deephackmode,dc=io

# cluster-admins, groups, deephackmode.io
dn: cn=cluster-admins,ou=groups,dc=deephackmode,dc=io
objectClass: groupOfNames
objectClass: top
cn: cluster-admins
description: pks.clusters.admin
member: cn=alexisv,ou=users,dc=deephackmode,dc=io

# cluster-managers, groups, deephackmode.io
dn: cn=cluster-managers,ou=groups,dc=deephackmode,dc=io
objectClass: groupOfNames
objectClass: top
cn: cluster-managers
description: pks.clusters.manage
member: cn=alexisv,ou=users,dc=deephackmode,dc=io

# cluster-devs, groups, deephackmode.io
dn: cn=cluster-devs,ou=groups,dc=deephackmode,dc=io
objectClass: groupOfNames
objectClass: top
cn: cluster-devs
description: developers
member: cn=alexisv,ou=users,dc=deephackmode,dc=io

EOF
{% endcapture %}

{% include codeblock.html code=ldif_code %}


Run `ldapsearch` with filter for a specific user record:

```
ubuntu@numbat:~$ env LDAPTLS_CACERT=ldap.deephackmode.io.crt ldapsearch -H 'ldaps://ldap.deephackmode.io' -b ou=users,dc=deephackmode,dc=io -D "cn=admin,dc=deephackmode,dc=io" -w admin cn=alexisv
# extended LDIF
#
# LDAPv3
# base <ou=users,dc=deephackmode,dc=io> with scope subtree
# filter: cn=alexisv
# requesting: ALL
#

# alexisv, users, deephackmode.io
dn: cn=alexisv,ou=users,dc=deephackmode,dc=io
objectClass: simpleSecurityObject
objectClass: inetOrgPerson
objectClass: posixAccount
cn: alexisv
sn: Villalon
mail: alexisv@deephackmode.io
description: alexisv
userPassword:: e1NTSEF9RC9IZ2FDV2t6SG03alVxRVl1SmJ1QVRWNVBBdTV6NXM=
uid: alexisv
uidNumber: 1000
gidNumber: 1000
homeDirectory: /home/alexisv

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
ubuntu@numbat:~$
```

Run `ldapsearch` to retrieve all groups with a specific member, and only output the `cn`:
```
ubuntu@numbat:~$ env LDAPTLS_CACERT=ldap.deephackmode.io.crt ldapsearch -H 'ldaps://ldap.deephackmode.io' -b ou=groups,dc=deephackmode,dc=io -D "cn=admin,dc=deephackmode,dc=io" -w admin 'member=cn=alexisv,ou=users,dc=deephackmode,dc=io' cn
# extended LDIF
#
# LDAPv3
# base <ou=groups,dc=deephackmode,dc=io> with scope subtree
# filter: member=cn=alexisv,ou=users,dc=deephackmode,dc=io
# requesting: cn
#

# tmc-admins, groups, deephackmode.io
dn: cn=tmc-admins,ou=groups,dc=deephackmode,dc=io
cn: tmc-admins

# tmc-members, groups, deephackmode.io
dn: cn=tmc-members,ou=groups,dc=deephackmode,dc=io
cn: tmc-members

# cluster-devs, groups, deephackmode.io
dn: cn=cluster-devs,ou=groups,dc=deephackmode,dc=io
cn: cluster-devs

# cluster-admins, groups, deephackmode.io
dn: cn=cluster-admins,ou=groups,dc=deephackmode,dc=io
cn: cluster-admins

# cluster-managers, groups, deephackmode.io
dn: cn=cluster-managers,ou=groups,dc=deephackmode,dc=io
cn: cluster-managers

# search result
search: 2
result: 0 Success

# numResponses: 6
# numEntries: 5
ubuntu@numbat:~$
```


At this point, the LDAP server now has the users and groups that I can use.  It also can communicate via TLS/SSL.  This should now be ready for the TMC-SM installation.


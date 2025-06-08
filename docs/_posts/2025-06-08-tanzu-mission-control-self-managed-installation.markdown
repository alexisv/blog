---
layout: post
title:  "Tanzu Mission Control Self-Managed Installation"
date:   2025-06-08 16:47:00 -0400
categories: deephackmode.io update
---
Now that I have a TKGM workload cluster prepared for Tanzu Mission Control Self-Managed (TMC-SM) installation, and a LDAP server up & running as a kubernetes workload in the TKGI cluster as well, the next thing to do is to install the TMC-SM package in the TKGM workload cluster.

As of this writing, the latest version of TMC-SM is 1.4.1, which I will use here.

### Add the tanzu-standard package repository

This is to the repository where tanzu packages will be downloaded from.  Since I have an airgapped environment, I need to specify the private Harbor registry URL, which is `harbor.deephackmode.io/tkg`.  

Also, I also need to specify the Tanzu Standarad Package Repo version, which is `v2024.8.21`.  To know which version to use according to the TKGM version, review [the version mapping table from the TKGM docs](https://techdocs.broadcom.com/us/en/vmware-tanzu/standalone-components/tanzu-kubernetes-grid/2-5/tkg/about-tkg-plugins.html#:~:text=The%20Tanzu%20Standard%20package%20repository%20version%20that%20works%20with%20the%20TKG%20version){:target="_blank"}.

Make sure that the current cluster context is set to the TKGM workload cluster.  Run the `tanzu package` command, with the right parameters, to add the repo.  Remember that I've been using the aliases `t` and `k` for the commands `tanzu` and `kubectl` respectively.

{% capture tpackage_code %}k config use-context av-wkld-admin@av-wkld
t package repository add tanzu-standard \
 --url harbor.deephackmode.io/tkg/packages/standard/repo:v2024.8.21 \
 --namespace tkg-system"
{% endcapture %}
{% include codeblock.html code=tpackage_code %}

Once added, it should now show up in the list of added repos.

```
ubuntu@numbat:~$ t package repository list -A

  NAMESPACE   NAME            SOURCE                                                                 STATUS
  tkg-system  tanzu-standard  (imgpkg) harbor.deephackmode.io/tkg/packages/standard/repo:v2024.8.21  Reconcile succeeded
ubuntu@numbat:~$
```

### Install cert-manager

The cert-manager workload will be installed to manage the certificates used in TMC-SM.

Set some environment variables that will be used for the cert-manager installation

{% capture setenvars_code %}export PACKAGE_USER_DEFINED_NAME="cert-manager"
export TANZU_PACKAGE_NAME="cert-manager.tanzu.vmware.com"
export TANZU_KUBECONFIG=~/.kube/config
export TANZU_PACKAGE_NAMESPACE="tkg-system"
export TANZU_PACKAGE_VALUES_FILE=cert-manager.tanzu.vmware.com-data-values.yaml
export TANZU_PACKAGE_VERSION=$(tanzu package available list "${TANZU_PACKAGE_NAME}" -n tkg-system -o json | jq .[].version | tail -n 1 | tr -d '"' )
{% endcapture %}
{% include codeblock.html code=setenvars_code %}

Make sure that the $TANZU_PACKAGE_VERSION has a valid value.

{% capture echoenvars_code %}echo $TANZU_PACKAGE_VERSION
{% endcapture %}
{% include codeblock.html code=echoenvars_code %}

Create the data-values YAML file to be used.

{% capture datavalues_code %}cat > cert-manager.tanzu.vmware.com-data-values.yaml <<EOF
---
#! The namespace in which to deploy cert-manager.
namespace: cert-manager

EOF
{% endcapture %}
{% include codeblock.html code=datavalues_code %}

Install the cert-manager package.

{% capture certmanagerinstall_code %}tanzu package install "${PACKAGE_USER_DEFINED_NAME}" \
  --package "${TANZU_PACKAGE_NAME}" \
  --version "${TANZU_PACKAGE_VERSION}" \
  --values-file "${TANZU_PACKAGE_VALUES_FILE}" \
  --namespace "${TANZU_PACKAGE_NAMESPACE}" \
  --kubeconfig "${TANZU_KUBECONFIG}"
{% endcapture %}
{% include codeblock.html code=certmanagerinstall_code %}


### Create the TMC-SM Cluster Issuer

The Cluster Issuer will be the self-signed certificate that will be used in the installation.  This will be managed by the cert-manager.

Create the definition YAML of the Cluster Issuer.

{% capture clusterissuer_code %}cat > tmcsm-cluster-issuer.yaml <<EOF
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: tmcsm-issuer
spec:
  ca:
    secretName: tmcsm-issuer
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: tmcsm-issuer
  namespace: cert-manager
spec:
  isCA: true
  commonName: tmcsm
  secretName: tmcsm-issuer
  issuerRef:
    name: selfsigned-issuer
    kind: ClusterIssuer
    group: cert-manager.io

EOF
{% endcapture %}
{% include codeblock.html code=clusterissuer_code %}

Create the cluster-issuer resource.

{% capture createclusterissuer_code %}kubectl apply -f tmcsm-cluster-issuer.yaml
{% endcapture %}
{% include codeblock.html code=createclusterissuer_code %}

Retrieve the self-signed cert (cluster-issuer) PEM data and save it in a file.  This file will contain the CA's that we want the cluster nodes to trust.  These CA certs will be added to the TMC-SM data values YAML file that will be created in an upcoming step.

{% capture getsecret_code %}kubectl get secret -n cert-manager tmcsm-issuer \
 -o=jsonpath="{.data.ca\.crt}" \
 | base64 -d > $HOME/trusted-ca.pem
{% endcapture %}
{% include codeblock.html code=getsecret_code %}

Also need to include the issuer of the Private Harbor server cert.  In this case, [that cert has been added](http://127.0.0.1:4000/deephackmode.io/update/2025/05/13/creating-management-cluster.html#set-up-the-docker-env){:target="_blank"} in `/usr/local/share/ca-certificates/opsman-ca.crt` earlier. 

{% capture getopsmancert_code %}cat /usr/local/share/ca-certificates/opsman-ca.crt >> ~/trusted-ca.pem
{% endcapture %}
{% include codeblock.html code=getopsmancert_code %}

Check the contents of `trusted-ca.pem` file, and make sure it has two certs (one is the cluster-issuer and the other is the CA for the private registry).

### Create the TMC-SM data values YAML file

This template worked for me.

{% capture tmcsmdv_code %}cat > $HOME/tmcsm-values.yaml <<EOF
clusterIssuer: "tmcsm-issuer"
contourEnvoy:
  serviceType: "LoadBalancer"
dnsZone: "tmc.deephackmode.io"
harborProject: "harbor.deephackmode.io/tmc-sm-automation"
minio:
  password: "Admin!23"
  username: "root"
authenticationType: ldap
oidc:
    issuerType: "pinniped"
    issuerURL: "https://pinniped-supervisor.tmc.deephackmode.io/provider/pinniped"
idpGroupRoles:
    admin: tmc-admins
    member: tmc-members
ldap:
    type: "ldap"
    host: "ldap.deephackmode.io"
    username: "cn=admin,dc=deephackmode,dc=io"
    password: "admin"
    domainName: "in-cluster-openldap"
    userBaseDN: "ou=users,dc=deephackmode,dc=io"
    userSearchFilter: "(&(objectClass=simpleSecurityObject)(cn={}))"
    userSearchAttributeUsername: cn
    groupBaseDN: "ou=groups,dc=deephackmode,dc=io"
    groupSearchFilter: "(&(objectClass=groupOfNames)(member={}))"
    rootCA: |
      -----BEGIN CERTIFICATE-----
      MIIEFTCCAv2gAwIBAgIUZGBjrHqE6+CI/TzWz2sBdh6deZEwDQYJKoZIhvcNAQEL
      BQAwgYsxCzAJBgNVBAYTAlVTMRMwEQYDVQQIDApDYWxpZm9ybmlhMRYwFAYDVQQH
      DA1TYW4gRnJhbmNpc2NvMRowGAYDVQQKDBFEZWVwIEhhY2ttb2RlIEluYzEUMBIG
      A1UECwwLRW5naW5lZXJpbmcxHTAbBgNVBAMMFGxkYXAuZGVlcGhhY2ttb2RlLmlv
      MB4XDTI1MDYwMTE4NDg0MloXDTI2MDYwMTE4NDg0MlowgYsxCzAJBgNVBAYTAlVT
      MRMwEQYDVQQIDApDYWxpZm9ybmlhMRYwFAYDVQQHDA1TYW4gRnJhbmNpc2NvMRow
      GAYDVQQKDBFEZWVwIEhhY2ttb2RlIEluYzEUMBIGA1UECwwLRW5naW5lZXJpbmcx
      HTAbBgNVBAMMFGxkYXAuZGVlcGhhY2ttb2RlLmlvMIIBIjANBgkqhkiG9w0BAQEF
      AAOCAQ8AMIIBCgKCAQEAoX0/oDhgjvSEC5QeeclWPGFWOSYjK9Nw7zyOuJ2Ue+hr
      fEs1rVDovErJZl0TSRTzmRlROAtw5CgINPD0htYPSc2LHBiL2/XLRfBzgSTFwBmw
      6O8OwT8ViDmeu6nP9t1xNXIs3bcwcywOtTS+9RUInkkR9JXZnC7SxjcOS535RKsd
      K4PYmKwJE2k2E1G/4MKeKYTR1zW/6AxluV7LAxj8sJjfuLdFeDNh+qm3Q1ylcwHF
      qb/T30spdU7+dQInd4oB+vvgnSKIqwQcfdvpIoCVGUK1OMod3yE9LcCzeXvc6u8C
      brkzZfZ5ObS1eAbYy5h5ql4PLtUGzHIhpRXTbj+lAQIDAQABo28wbTAfBgNVHREE
      GDAWghRsZGFwLmRlZXBoYWNrbW9kZS5pbzAJBgNVHRMEAjAAMAsGA1UdDwQEAwIF
      oDATBgNVHSUEDDAKBggrBgEFBQcDATAdBgNVHQ4EFgQUEVkMaia9zkgvvqruuhe1
      vvNGzvkwDQYJKoZIhvcNAQELBQADggEBAJxG396p4bemfj3nDIG8ZAhSIFNPufdd
      6izg9yus7VKWImI/aCZizkYzJBEb2w+J2ZgHciPI8+UWPGI/Rs0yh60Gmb21Kfo3
      tf83Nk4Xi6WncPvrWoENHmMI8cF1bwqLOveEHQ4PefUHil0mRvRm//5HXYBBXkhk
      QHRb/mcf5yds6Yrt1v1tM8DespzipYazcF0cSln8BZLOCoGHbLSkQRf24W6ooaTV
      qqDUljP1G2lksgjxyU7tsH0E1noJnXTvDPmv/DVWLispc6HwPy9+pRNA48aqQbWJ
      ObqgRFoGu4r8pTWRG5o/CI+Npv5Nsv7qZ9LiUrlNxjVzlReHV4JGEGU=
      -----END CERTIFICATE-----
postgres:
  userPassword: "Admin!23"
  maxConnections: 300
telemetry:
  ceipAgreement: false
  ceipOptIn: false
  eanNumber: ""
trustedCAs:
  trusted-ca.pem: |-
    -----BEGIN CERTIFICATE-----
    MIIC7DCCAdSgAwIBAgIQXB7ak1h7KzrTn1EVGOS60TANBgkqhkiG9w0BAQsFADAQ
    MQ4wDAYDVQQDEwV0bWNzbTAeFw0yNTA2MDExNDM2MjNaFw0yNTA4MzAxNDM2MjNa
    MBAxDjAMBgNVBAMTBXRtY3NtMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKC
    AQEAxAfMZLctyYSqjrxb95D8ar9iWSFV6sylZQpZvR9AuEr2v8xgA0ztmDYhvLnq
    FGrtuIc1e3+wMk4ZHVdkI+20EsE+h3Ymz3n8t0vBBaaFUD1cILQ62cCjqwjGDWFY
    yvoFOePskWf/RpVhylv2TTTSPBs/clFpLwO0Fs0cD2pxqgHTabSpxWujvgqYFsTJ
    epvA1lBz9y1rFwyw1d/4BqEZb1koKhfkwqBFjWoFaksrselV3IxxQ3Nh1qoX4foP
    Rotk2kCGvWA8xOQfH4lpmUPnLS9Wuz/I38ZlO8GvAmcTygWe8BKZdIOpvk0QSdlc
    F3aXFx95A8LM1uJ8D0qfBewg/QIDAQABo0IwQDAOBgNVHQ8BAf8EBAMCAqQwDwYD
    VR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQUbBdkx1kTmvtfWgO6DFJ/SoM3xD8wDQYJ
    KoZIhvcNAQELBQADggEBADKlQyNIzz0JjcpFTp2er+aw8dQ+LhkkHpx0TCgCSCA8
    bvALZ5LSflo0dUmwgy7GGppw50spiSfEHx189+Va+1OAEJqtRplO+WX/3sijsJLn
    PmOk7xM6asYh83067OKvbNQGcMoA0HswOiF1CWKMoTeiVW00+t42EjxsYYBX6Rov
    vn06aTdyAKZuIAu3abWTchssQ4+8nE1/T56o9U7fS2Jd/79qdskHi+OGBTweFHhU
    uTgCqYFjcDXdPArLjQTguZl4/HjdnMo2f1Nacm0x5ypw8ikpaXN5ANgOW302qYZI
    dgUod7HpQ36+eihcuQEbD6sjReMAiMd3xnGifgT2u0U=
    -----END CERTIFICATE-----
    -----BEGIN CERTIFICATE-----
    MIIDUDCCAjigAwIBAgIULOZPbDD4cjaev999fD5CDO8vyy4wDQYJKoZIhvcNAQEL
    BQAwHzELMAkGA1UEBhMCVVMxEDAOBgNVBAoMB1Bpdm90YWwwHhcNMjUwNDA5MDA0
    MjUyWhcNMjkwNDA5MDA0MjUyWjAfMQswCQYDVQQGEwJVUzEQMA4GA1UECgwHUGl2
    b3RhbDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMM2LDuVDtmBOKXa
    tS0dC898Zi5ANXVo6+iplVo0un1HRPtEJIQD+i7GwD3LgrUFSVk+95FnK1Q/sH5n
    f22YByrSnAgAyMNU2ePNwFpMI13wMEgXkn98PiNUJleL/h3+ZlOCktLskEXHM+lv
    hKCRg4O3JEVM9AqAmIfhJGSs832VzBtcJfRQ4WGsluD3u2DCly8UNo4yYfZMfW+y
    h086ma7b4Bi68QosvrcIm/TMj1pi51zn2uExmgAWlXORDuaLRpFa2qJTLJaGj743
    MlQAy7ozuma0MFQ2C2h0E8HJokjKsRZGUVn/Ha09Ys4Oj8H+QUAgC625t8RxW083
    N4ILwH0CAwEAAaOBgzCBgDAdBgNVHQ4EFgQUE/izKiTagXr9a3l2MfSpqWkz2gIw
    HwYDVR0jBBgwFoAUE/izKiTagXr9a3l2MfSpqWkz2gIwHQYDVR0lBBYwFAYIKwYB
    BQUHAwIGCCsGAQUFBwMBMA8GA1UdEwEB/wQFMAMBAf8wDgYDVR0PAQH/BAQDAgEG
    MA0GCSqGSIb3DQEBCwUAA4IBAQCh28ZuiWlG0wzl908zGaM9B4J3SMxdKYXDRlWr
    CQe/m7qq1s7S/VoO3dXRSY4JDrbksB1mBbqHtXwMaA8NvaKEmgjwdoEzHO+zhDsr
    /fusAuEGPZ9AGqvKpItki06m1tyqC0PS+/wBXC5yfUm1SGiGZgpeRbh5gL4o+LTP
    TKuygzCZwpPaZSeF3D2ePmom/OnojisEIf4e7vR6bfNUyo0PBzGfz2z/GuUf0MO8
    s7yIUpHm1o9YVnozihCRbpQLJTCDClvQDpZksW6RIf6ZpcnWHxo6n29YZxm66ScC
    xM+j+j+RUPE2+I1LeRhuDUWcxpCe/l/o6zlo1IRatsuDmDDF
    -----END CERTIFICATE-----
EOF
{% endcapture %}
{% include codeblock.html code=tmcsmdv_code %}

Make sure that the certificates are updated.  The `.ldap.rootCA` should have the PEM data from the `ldap.deephackmode.io.crt` file.  The `trustedCAs.trusted-ca.pem` should have the contents for `$HOME/trusted-ca.pem`.

#### Use yq to make adding/changing certs in the YAML easier

You can install `yq` as below:

{% capture installyq_code %}yq_url="https://github.com/mikefarah/yq/releases/download/v4.6.1/yq_linux_amd64"
wget -nv ${yq_url}
sudo install yq_linux_amd64 /usr/local/bin/yq
{% endcapture %}
{% include codeblock.html code=installyq_code %}

Then, use `yq` to replace the value of `.trustedCAs.trusted-ca.pem` in the `tmcsm-values.yaml` file with the contents of `$HOME/trusted-ca.pem`.


{% capture useyq_code %}TRUSTED_CERT="$(cat $HOME/trusted-ca.pem)" yq e \
 '.trustedCAs."trusted-ca.pem" = strenv(TRUSTED_CERT) \
 | .trustedCAs."trusted-ca.pem" style="literal"' \
 -i $HOME/tmcsm-values.yaml
{% endcapture %}
{% include codeblock.html code=useyq_code %}

### Download the tmc-sm bundle from the Broadcom Support Portal.

Go to the Broadcom Support Portal, and download the v1.4.1 of the TMC-SM Bundle.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-06-08-tanzu-mission-control-self-managed-installation/tmcsm-download.png" alt="Download TMC-SM bundle" title="Download TMC-SM bundle">
</div>
<figcaption>Download TMC-SM bundle</figcaption>
</figure> 

Create a `tanzumc` directory and untar the contents of the downloaded file there.

```
ubuntu@numbat:/mnt/tmp/ubuntu$ ls -l
total 6729684
-rw-r--r-- 1 ubuntu ubuntu 6891190784 Jun  7 22:32 tmc_self_managed_1.4.1.tar
ubuntu@numbat:/mnt/tmp/ubuntu$ mkdir tanzumc
ubuntu@numbat:/mnt/tmp/ubuntu$ ls -l
total 6729688
drwxrwxr-x 6 ubuntu ubuntu       4096 Jun  8 03:17 tanzumc
-rw-r--r-- 1 ubuntu ubuntu 6891190784 Jun  7 22:32 tmc_self_managed_1.4.1.tar
ubuntu@numbat:/mnt/tmp/ubuntu$ tar -xf ./tmc_self_managed_1.4.1.tar -C ./tanzumc
ubuntu@numbat:/mnt/tmp/ubuntu$
```

### Push the TMC-SM images to the private registry

In the Harbor private registry, create a new project named "tmc-sm-automation".  This is where the TMC-SM images will be pushed and stored.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-06-08-tanzu-mission-control-self-managed-installation/harbor-tmc-sm-project.png" alt="Create new Harbor project" title="Create new Harbor project">
</div>
<figcaption>Create new Harbor project</figcaption>
</figure> 

In the jumpbox, login to the private registry.

{% capture dockerlogin_code %}docker login harbor.deephackmode.io
{% endcapture %}
{% include codeblock.html code=dockerlogin_code %}

Change directory to the parent directory of the `tanzumc` directory that was create earlier.

{% capture cd_code %}cd /mnt/tmp/ubuntu
{% endcapture %}
{% include codeblock.html code=cd_code %}

```
ubuntu@numbat:/mnt/tmp/ubuntu$ ls -l
total 6729688
drwxrwxr-x 6 ubuntu ubuntu       4096 Jun  8 03:17 tanzumc
-rw-r--r-- 1 ubuntu ubuntu 6891190784 Jun  7 22:32 tmc_self_managed_1.4.1.tar
ubuntu@numbat:/mnt/tmp/ubuntu$
```

Push the TMC-SM images to the registry.

{% capture push_code %}tanzumc/tmc-sm push-images harbor \
 --project harbor.deephackmode.io/tmc-sm-automation \
 --username admin \
 --password 'pa$$w0rd'
{% endcapture %}
{% include codeblock.html code=push_code %}

After it completes, the below output should be shown.

```
...

Image Staging Complete. Next Steps:
Setup Kubeconfig (if not already done) to point to cluster:
export KUBECONFIG={YOUR_KUBECONFIG}

Create 'tmc-local' namespace: kubectl create namespace tmc-local

Download Tanzu CLI from Customer Connect (If not already installed)

Update TMC Self Managed Package Repository:
Run: tanzu package repository add tanzu-mission-control-packages --url "harbor.deephackmode.io/tmc-sm-automation/package-repository:1.4.1" --namespace tmc-local

Create a values based on the TMC Self Managed Package Schema:
View the Values Schema: tanzu package available get "tmc.tanzu.vmware.com/1.4.1" --namespace tmc-local --values-schema
Create a Values file named values.yaml matching the schema

Install the TMC Self Managed Package:
Run: tanzu package install tanzu-mission-control -p tmc.tanzu.vmware.com --version "1.4.1" --values-file values.yaml --namespace tmc-local
```


### Create the tmc-local namespace

Let's create the namespace where the tmc-sm workload app will live.

{% capture ns_code %}k create ns tmc-local
{% endcapture %}
{% include codeblock.html code=ns_code %}

### Add the TMC-SM package repo

Run the command to add the TMC-SM package repo to the cluster.

{% capture tmcsmpkgr_code %}tanzu package repository add tanzu-mission-control-packages \
  --url "harbor.deephackmode.io/tmc-sm-automation/package-repository:1.4.1"\
  --namespace tmc-local
{% endcapture %}
{% include codeblock.html code=tmcsmpkgr_code %}

### Install the TMC-SM Package

Run the command to install the TMC-SM v1.4.1 package.
{% capture tmcsmpkginstall_code %}tanzu package install tanzu-mission-control \
  -p tmc.tanzu.vmware.com \
  --version "1.4.1" \
  --values-file tmcsm-values.yaml \
  --namespace tmc-local
{% endcapture %}
{% include codeblock.html code=tmcsmpkginstall_code %}


### Verify if installation was successful

Check if the packages are all reconciled successfully.
```
ubuntu@numbat:~$ t package installed list -A

  NAMESPACE   NAME                                       PACKAGE-NAME                                        PACKAGE-VERSION        STATUS             
  tkg-system  av-wkld-antrea                             antrea.tanzu.vmware.com                             1.13.3+vmware.4-tkg.1  Reconcile succeeded
  tkg-system  av-wkld-capabilities                       capabilities.tanzu.vmware.com                       0.32.3+vmware.1        Reconcile succeeded
  tkg-system  av-wkld-gateway-api                        gateway-api.tanzu.vmware.com                        1.0.0+vmware.1-tkg.2   Reconcile succeeded
  tkg-system  av-wkld-load-balancer-and-ingress-service  load-balancer-and-ingress-service.tanzu.vmware.com  1.12.2+vmware.1-tkg.1  Reconcile succeeded
  tkg-system  av-wkld-metrics-server                     metrics-server.tanzu.vmware.com                     0.6.2+vmware.8-tkg.1   Reconcile succeeded
  tkg-system  av-wkld-pinniped                           pinniped.tanzu.vmware.com                           0.25.0+vmware.3-tkg.1  Reconcile succeeded
  tkg-system  av-wkld-secretgen-controller               secretgen-controller.tanzu.vmware.com               0.15.0+vmware.2-tkg.1  Reconcile succeeded
  tkg-system  av-wkld-tkg-storageclass                   tkg-storageclass.tanzu.vmware.com                   0.32.3+vmware.1        Reconcile succeeded
  tkg-system  av-wkld-vsphere-cpi                        vsphere-cpi.tanzu.vmware.com                        1.28.0+vmware.3-tkg.2  Reconcile succeeded
  tkg-system  av-wkld-vsphere-csi                        vsphere-csi.tanzu.vmware.com                        3.3.0+vmware.1-tkg.1   Reconcile succeeded
  tkg-system  cert-manager                               cert-manager.tanzu.vmware.com                       1.7.2+vmware.3-tkg.3   Reconcile succeeded
  tmc-local   contour                                    contour.bitnami.com                                 18.2.19                Reconcile succeeded
  tmc-local   kafka                                      kafka.bitnami.com                                   28.3.2                 Reconcile succeeded
  tmc-local   kafka-topic-controller                     kafka-topic-controller.tmc.tanzu.vmware.com         0.0.33                 Reconcile succeeded
  tmc-local   minio                                      minio.bitnami.com                                   14.6.8                 Reconcile succeeded
  tmc-local   pinniped                                   pinniped.bitnami.com                                2.3.1                  Reconcile succeeded
  tmc-local   postgres                                   tmc-local-postgres.tmc.tanzu.vmware.com             0.0.138                Reconcile succeeded
  tmc-local   postgres-endpoint-controller               postgres-endpoint-controller.tmc.tanzu.vmware.com   0.1.72                 Reconcile succeeded
  tmc-local   redis                                      redis.bitnami.com                                   19.5.15                Reconcile succeeded
  tmc-local   reloader-reloader                          reloader-reloader.tmc.tanzu.vmware.com              1.0.107                Reconcile succeeded
  tmc-local   s3-access-operator                         s3-access-operator.tmc.tanzu.vmware.com             0.1.36                 Reconcile succeeded
  tmc-local   tanzu-mission-control                      tmc.tanzu.vmware.com                                1.4.1                  Reconcile succeeded
  tmc-local   tmc-local-monitoring                       monitoring.tmc.tanzu.vmware.com                     0.0.22                 Reconcile succeeded
  tmc-local   tmc-local-stack                            tmc-local-stack.tmc.tanzu.vmware.com                0.1.1865088            Reconcile succeeded
  tmc-local   tmc-local-stack-secrets                    tmc-local-stack-secrets.tmc.tanzu.vmware.com        0.1.1865088            Reconcile succeeded
  tmc-local   tmc-local-support                          tmc-local-support.tmc.tanzu.vmware.com              0.1.1865088            Reconcile succeeded
ubuntu@numbat:~$
```

Check if all pods are running fine or completed successfully.
```
ubuntu@numbat:~$ k -n tmc-local get po
NAME                                                 READY   STATUS      RESTARTS      AGE
account-manager-server-dc57b5544-f8vpm               1/1     Running     0             18h
account-manager-server-dc57b5544-k8nxz               1/1     Running     0             18h
agent-gateway-server-76d6456674-2bkvn                1/1     Running     0             18h
agent-gateway-server-76d6456674-j5jlq                1/1     Running     0             18h
alertmanager-tmc-local-monitoring-tmc-local-0        2/2     Running     0             18h
api-gateway-server-75f6cd4cd6-sc5mc                  1/1     Running     0             18h
api-gateway-server-75f6cd4cd6-xxfwb                  1/1     Running     0             18h
audit-service-consumer-77d7d6f4db-95qr2              1/1     Running     0             18h
audit-service-consumer-77d7d6f4db-q9fmd              1/1     Running     0             18h
audit-service-server-68c9fbcfd-6ccml                 1/1     Running     0             18h
audit-service-server-68c9fbcfd-w6dzn                 1/1     Running     0             18h
auth-manager-server-746486c99-27xzp                  1/1     Running     0             18h
auth-manager-server-746486c99-pnjvr                  1/1     Running     0             18h
authentication-server-7dc45c8f4-k6jnx                1/1     Running     0             18h
authentication-server-7dc45c8f4-pmmr4                1/1     Running     0             18h
cluster-agent-service-server-5c744cd5d5-j8kgx        1/1     Running     0             18h
cluster-agent-service-server-5c744cd5d5-k9522        1/1     Running     0             18h
cluster-config-server-6bfbf674d6-rncxg               1/1     Running     0             18h
cluster-config-server-6bfbf674d6-tzqtf               1/1     Running     0             18h
cluster-object-service-server-6d489f8d8-rx7zh        1/1     Running     0             18h
cluster-object-service-server-6d489f8d8-s5gkg        1/1     Running     0             18h
cluster-reaper-server-57488ddfc6-8jjjp               1/1     Running     0             18h
cluster-secret-server-698d4cdb5f-rv6dv               1/1     Running     0             18h
cluster-secret-server-698d4cdb5f-s52dv               1/1     Running     0             18h
cluster-service-server-74dc699659-4wkrg              1/1     Running     0             18h
cluster-service-server-74dc699659-wm7k7              1/1     Running     0             18h
cluster-sync-egest-6ff58c7b7b-bvk47                  1/1     Running     0             18h
cluster-sync-egest-6ff58c7b7b-fnlwz                  1/1     Running     0             18h
cluster-sync-ingest-7c7f57d854-2kpd6                 1/1     Running     0             18h
cluster-sync-ingest-7c7f57d854-fgqpc                 1/1     Running     0             18h
contour-contour-779554fd5d-x2z8f                     1/1     Running     0             18h
contour-contour-certgen-qljhj                        0/1     Completed   0             18h
contour-envoy-5k9d7                                  2/2     Running     0             18h
contour-envoy-tznrz                                  2/2     Running     0             18h
contour-envoy-zkm49                                  2/2     Running     0             18h
dataprotection-server-545d85567f-7vrl7               1/1     Running     0             18h
dataprotection-server-545d85567f-g66hq               1/1     Running     0             18h
events-service-consumer-745c4f96b-9kj5h              1/1     Running     0             18h
events-service-consumer-745c4f96b-gfltt              1/1     Running     0             18h
events-service-server-68c69f8677-fdtxg               1/1     Running     0             18h
events-service-server-68c69f8677-sxpz2               1/1     Running     0             18h
fanout-service-server-b9c478955-pgm6d                1/1     Running     0             18h
fanout-service-server-b9c478955-vnvwk                1/1     Running     0             18h
feature-flag-service-server-7764b6dd9d-tw6pr         1/1     Running     0             18h
helm-deployment-server-679b89bdd6-hkg55              1/1     Running     0             18h
helm-deployment-server-679b89bdd6-pl7qt              1/1     Running     0             18h
inspection-server-5cf8678bb-b7544                    2/2     Running     0             18h
inspection-server-5cf8678bb-crvpl                    2/2     Running     0             18h
intent-server-6cf59cb6d7-phnb2                       1/1     Running     0             18h
intent-server-6cf59cb6d7-pqlz7                       1/1     Running     0             18h
kafka-controller-0                                   1/1     Running     0             18h
kafka-exporter-5f6f84d484-4qfmz                      1/1     Running     0             18h
kafka-topic-controller-7c499956f5-pplv2              1/1     Running     0             18h
kafka-zookeeper-0                                    1/1     Running     0             18h
landing-service-server-6d88bcfdc8-9h4cr              1/1     Running     0             18h
minio-7554bbdc98-6rn97                               1/1     Running     1 (18h ago)   18h
minio-provisioning-prv5l                             0/1     Completed   0             18h
onboarding-service-server-7f774d6dd7-twhnk           1/1     Running     0             18h
onboarding-service-server-7f774d6dd7-zhhkv           1/1     Running     0             18h
package-deployment-server-9f5848c84-cxv4n            1/1     Running     0             18h
package-deployment-server-9f5848c84-rfn55            1/1     Running     0             18h
pinniped-supervisor-684cdd9d77-9tb5v                 1/1     Running     0             18h
policy-engine-server-6494bdf78-54znw                 1/1     Running     0             18h
policy-engine-server-6494bdf78-5rc9v                 1/1     Running     0             18h
policy-insights-server-6d749bf9bc-5xgd6              1/1     Running     0             18h
policy-insights-server-6d749bf9bc-8rxzd              1/1     Running     0             18h
policy-sync-service-server-85b44b8fd5-m56np          1/1     Running     0             18h
policy-view-service-server-78959-pkx92               1/1     Running     0             18h
policy-view-service-server-78959-r57xp               1/1     Running     0             18h
postgres-endpoint-controller-6778b889fc-rwksm        1/1     Running     0             18h
postgres-postgresql-0                                2/2     Running     0             18h
prometheus-server-tmc-local-monitoring-tmc-local-0   2/2     Running     0             18h
provisioner-service-server-8457795bd-glbzk           1/1     Running     0             18h
provisioner-service-server-8457795bd-k6ddw           1/1     Running     0             18h
rate-limit-service-server-78676cb956-m58gd           3/3     Running     0             18h
rate-limit-service-server-78676cb956-xp9pd           3/3     Running     0             18h
redis-master-0                                       1/1     Running     0             18h
reloader-reloader-674f8bb9f-rvhxp                    1/1     Running     0             18h
resource-manager-server-7bfcfbbf6b-56r4n             1/1     Running     0             18h
resource-manager-server-7bfcfbbf6b-w6n6m             1/1     Running     0             18h
s3-access-operator-845c575b8f-d5hbz                  1/1     Running     0             18h
settings-service-server-bb548f955-6fx2r              1/1     Running     0             18h
settings-service-server-bb548f955-fkl95              1/1     Running     0             18h
telemetry-event-service-consumer-f68fdc459-4hvr9     1/1     Running     0             18h
telemetry-event-service-consumer-f68fdc459-zjblm     1/1     Running     0             18h
tenancy-service-server-7c8487d976-l2hkq              1/1     Running     0             18h
ui-server-698d774d88-8kdbq                           1/1     Running     0             18h
ui-server-698d774d88-rdlj9                           1/1     Running     0             18h
wcm-server-84b7dd6f46-8267j                          1/1     Running     0             18h
wcm-server-84b7dd6f46-zmmml                          1/1     Running     0             18h
ubuntu@numbat:~$
```

Check the services.  One of them is `contour-envoy` LoadBalancer and should have an ExternalIP.

```
ubuntu@numbat:~$ k get svc -n tmc-local
NAME                                               TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                      AGE
account-manager-grpc                               ClusterIP      100.64.146.56    <none>           443/TCP                      18h
account-manager-service                            ClusterIP      100.65.120.244   <none>           443/TCP,7777/TCP             18h
agent-gateway-service                              ClusterIP      100.66.241.163   <none>           443/TCP,8443/TCP,7777/TCP    18h
alertmanager-tmc-local-monitoring-tmc-local        ClusterIP      100.66.81.77     <none>           9093/TCP                     18h
api-gateway-service                                ClusterIP      100.69.62.101    <none>           443/TCP,8443/TCP,7777/TCP    18h
audit-service-consumer                             ClusterIP      100.65.93.247    <none>           7777/TCP                     18h
audit-service-grpc                                 ClusterIP      100.66.188.128   <none>           443/TCP                      18h
audit-service-rest                                 ClusterIP      100.69.215.235   <none>           443/TCP                      18h
audit-service-service                              ClusterIP      100.68.91.3      <none>           443/TCP,8443/TCP,7777/TCP    18h
auth-manager-server                                ClusterIP      100.67.60.71     <none>           443/TCP                      18h
auth-manager-service                               ClusterIP      100.65.73.199    <none>           443/TCP,7777/TCP             18h
authentication-grpc                                ClusterIP      100.69.189.3     <none>           443/TCP                      18h
authentication-service                             ClusterIP      100.71.0.141     <none>           443/TCP,7777/TCP             18h
cluster-agent-service-grpc                         ClusterIP      100.67.173.245   <none>           443/TCP                      18h
cluster-agent-service-installer                    ClusterIP      100.69.38.17     <none>           80/TCP                       18h
cluster-agent-service-service                      ClusterIP      100.65.29.169    <none>           443/TCP,80/TCP,7777/TCP      18h
cluster-config-service                             ClusterIP      100.70.224.60    <none>           443/TCP,7777/TCP             18h
cluster-object-service-grpc                        ClusterIP      100.67.48.198    <none>           443/TCP                      18h
cluster-object-service-service                     ClusterIP      100.70.3.105     <none>           443/TCP,8443/TCP,7777/TCP    18h
cluster-reaper-grpc                                ClusterIP      100.68.164.19    <none>           443/TCP                      18h
cluster-reaper-service                             ClusterIP      100.71.199.18    <none>           443/TCP,7777/TCP             18h
cluster-secret-service                             ClusterIP      100.69.127.223   <none>           443/TCP,7777/TCP             18h
cluster-service-grpc                               ClusterIP      100.71.29.227    <none>           443/TCP                      18h
cluster-service-rest                               ClusterIP      100.71.32.41     <none>           443/TCP                      18h
cluster-service-service                            ClusterIP      100.65.61.10     <none>           443/TCP,8443/TCP,7777/TCP    18h
cluster-sync-egest                                 ClusterIP      100.66.126.38    <none>           443/TCP,7777/TCP             18h
cluster-sync-egest-grpc                            ClusterIP      100.64.137.229   <none>           443/TCP                      18h
cluster-sync-ingest                                ClusterIP      100.64.76.250    <none>           443/TCP,7777/TCP             18h
cluster-sync-ingest-grpc                           ClusterIP      100.70.160.14    <none>           443/TCP                      18h
contour                                            ClusterIP      100.67.120.162   <none>           8001/TCP                     18h
contour-envoy                                      LoadBalancer   100.71.214.251   192.168.86.201   80:32218/TCP,443:32526/TCP   18h
contour-envoy-metrics                              ClusterIP      None             <none>           8002/TCP                     18h
dataprotection-grpc                                ClusterIP      100.69.4.237     <none>           443/TCP                      18h
dataprotection-service                             ClusterIP      100.71.90.191    <none>           443/TCP,8443/TCP,7777/TCP    18h
events-service-consumer                            ClusterIP      100.65.141.139   <none>           7777/TCP                     18h
events-service-grpc                                ClusterIP      100.68.193.33    <none>           443/TCP                      18h
events-service-service                             ClusterIP      100.65.10.31     <none>           443/TCP,7777/TCP             18h
fanout-service-grpc                                ClusterIP      100.68.221.61    <none>           443/TCP                      18h
fanout-service-service                             ClusterIP      100.65.126.220   <none>           443/TCP,7777/TCP             18h
feature-flag-service-grpc                          ClusterIP      100.71.137.124   <none>           443/TCP                      18h
feature-flag-service-service                       ClusterIP      100.64.254.99    <none>           443/TCP,7777/TCP             18h
helm-deployment-service                            ClusterIP      100.68.160.40    <none>           443/TCP,7777/TCP             18h
inspection-grpc                                    ClusterIP      100.70.131.111   <none>           443/TCP                      18h
inspection-service                                 ClusterIP      100.67.16.181    <none>           443/TCP,7777/TCP             18h
intent-grpc                                        ClusterIP      100.66.107.198   <none>           443/TCP                      18h
intent-service                                     ClusterIP      100.68.64.189    <none>           443/TCP,7777/TCP             18h
kafka                                              ClusterIP      100.69.51.174    <none>           9092/TCP                     18h
kafka-controller-headless                          ClusterIP      None             <none>           9094/TCP,9092/TCP,9093/TCP   18h
kafka-metrics                                      ClusterIP      100.65.71.22     <none>           9308/TCP                     18h
kafka-zookeeper                                    ClusterIP      100.68.77.20     <none>           2181/TCP                     18h
kafka-zookeeper-headless                           ClusterIP      None             <none>           2181/TCP,2888/TCP,3888/TCP   18h
landing-service-metrics                            ClusterIP      None             <none>           7777/TCP                     18h
landing-service-rest                               ClusterIP      100.65.191.234   <none>           443/TCP                      18h
landing-service-server                             ClusterIP      100.71.145.239   <none>           443/TCP                      18h
minio                                              ClusterIP      100.69.82.179    <none>           9000/TCP,9001/TCP            18h
onboarding-service-metrics                         ClusterIP      None             <none>           7777/TCP                     18h
onboarding-service-rest                            ClusterIP      100.71.152.81    <none>           443/TCP                      18h
package-deployment-eula-server                     ClusterIP      100.64.187.205   <none>           80/TCP                       18h
package-deployment-service                         ClusterIP      100.69.236.96    <none>           443/TCP,80/TCP,7777/TCP      18h
pinniped-supervisor                                ClusterIP      100.69.214.139   <none>           443/TCP                      18h
pinniped-supervisor-api                            ClusterIP      100.65.210.114   <none>           443/TCP                      18h
policy-engine-grpc                                 ClusterIP      100.66.250.27    <none>           443/TCP                      18h
policy-engine-service                              ClusterIP      100.71.196.235   <none>           443/TCP,7777/TCP             18h
policy-insights-grpc                               ClusterIP      100.69.80.157    <none>           443/TCP                      18h
policy-insights-service                            ClusterIP      100.70.219.250   <none>           443/TCP,7777/TCP             18h
policy-sync-service-service                        ClusterIP      100.67.223.9     <none>           7777/TCP                     18h
policy-view-service-grpc                           ClusterIP      100.66.47.232    <none>           443/TCP                      18h
policy-view-service-service                        ClusterIP      100.71.130.126   <none>           443/TCP,7777/TCP             18h
postgres-endpoint-controller                       ClusterIP      100.64.140.129   <none>           9876/TCP                     18h
postgres-postgresql                                ClusterIP      100.71.134.220   <none>           5432/TCP                     18h
postgres-postgresql-hl                             ClusterIP      None             <none>           5432/TCP                     18h
postgres-postgresql-metrics                        ClusterIP      100.70.73.186    <none>           9187/TCP                     18h
prometheus-server-tmc-local-monitoring-tmc-local   ClusterIP      100.64.10.31     <none>           9090/TCP                     18h
provisioner-service-grpc                           ClusterIP      100.69.59.4      <none>           443/TCP                      18h
provisioner-service-service                        ClusterIP      100.65.101.205   <none>           443/TCP,7777/TCP             18h
rate-limit-service-server                          ClusterIP      100.69.104.129   <none>           80/TCP,443/TCP,7777/TCP      18h
redis-headless                                     ClusterIP      None             <none>           6379/TCP                     18h
redis-master                                       ClusterIP      100.67.129.75    <none>           6379/TCP                     18h
resource-manager-grpc                              ClusterIP      100.66.45.99     <none>           443/TCP                      18h
resource-manager-service                           ClusterIP      100.65.114.43    <none>           443/TCP,8443/TCP,7777/TCP    18h
s3-access-operator                                 ClusterIP      100.64.161.115   <none>           443/TCP,8080/TCP             18h
settings-service-server                            ClusterIP      100.67.94.123    <none>           443/TCP,7777/TCP             18h
telemetry-event-service-consumer                   ClusterIP      100.64.194.201   <none>           7777/TCP                     18h
tenancy-service-metrics-headless                   ClusterIP      None             <none>           7777/TCP                     18h
tenancy-service-tenancy-service                    ClusterIP      100.68.203.68    <none>           443/TCP                      18h
tenancy-service-tenancy-service-rest               ClusterIP      100.70.26.183    <none>           443/TCP                      18h
ui-server                                          ClusterIP      100.66.124.148   <none>           8443/TCP,7777/TCP            18h
wcm-grpc                                           ClusterIP      100.69.212.118   <none>           443/TCP                      18h
wcm-service                                        ClusterIP      100.69.215.241   <none>           443/TCP,8443/TCP,7777/TCP    18h
ubuntu@numbat:~$
```

### Add the DNS entries for TMC-SM components

Find the service named contour and take not of the ExternalIP of the LoadBalancer.

```
contour-envoy                                      LoadBalancer   100.71.214.251   192.168.86.201   80:32218/TCP,443:32526/TCP   
```

In my case, I'm using dnsmasq and I added the following A records pointing to the LoadBalancer ExternalIP.
```
host-record=tmc.deephackmode.io,192.168.86.201
host-record=alertmanager.tmc.deephackmode.io,192.168.86.201
host-record=auth.tmc.deephackmode.io,192.168.86.201
host-record=blob.tmc.deephackmode.io,192.168.86.201
host-record=console.s3.tmc.deephackmode.io,192.168.86.201
host-record=gts-rest.tmc.deephackmode.io,192.168.86.201
host-record=gts.tmc.deephackmode.io,192.168.86.201
host-record=landing.tmc.deephackmode.io,192.168.86.201
host-record=pinniped-supervisor.tmc.deephackmode.io,192.168.86.201
host-record=prometheus.tmc.deephackmode.io,192.168.86.201
host-record=s3.tmc.deephackmode.io,192.168.86.201
host-record=tmc-local.s3.tmc.deephackmode.io,192.168.86.201
```

Reload the DNS configuration, and make sure that the TMC host names are resolving to the IP address accordingly.

### Log in to the TMC-SM UI

Go to https://tmc.deephackmode.io.  You should be greeted by this sign in page.

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-06-08-tanzu-mission-control-self-managed-installation/tmc-sm-sign-in-screen.png" alt="TMC-SM UI Sign In" title="TMC-SM UI Sign In">
</div>
<figcaption>TMC-SM UI Sign In</figcaption>
</figure> 

Click Sign In, and then you would be redirected to the Pinniped Login.

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-06-08-tanzu-mission-control-self-managed-installation/pinniped-login-screen.png" alt="Pinniped Login" title="Pinniped Login">
</div>
<figcaption>Pinniped Login</figcaption>
</figure> 

Login with your LDAP Username and Password.  Review the post [Deploy a LDAP server workload](https://deephackmode.io/deephackmode.io/update/2025/06/07/deploy-a-ldap-server-workload.html){:target="_blank"}, to know what username and password that can be used.

Once logged in, you should this screen.

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-06-08-tanzu-mission-control-self-managed-installation/tmc-sm-logged-in.png" alt="TMC-SM Logged In" title="TMC-SM Logged In">
</div>
<figcaption>TMC-SM Logged In</figcaption>
</figure> 

Switch to Dark Mode.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-06-08-tanzu-mission-control-self-managed-installation/tmc-sm-dark-mode.png" alt="TMC-SM Dark Mode" title="TMC-SM Dark Mode">
</div>
<figcaption>TMC-SM Dark Mode</figcaption>
</figure> 

The Clusters page is empty for now.

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-06-08-tanzu-mission-control-self-managed-installation/clusters-page.png" alt="Empty Clusters page" title="Empty Clusters page">
</div>
<figcaption>Empty Clusters page</figcaption>
</figure> 

Let's attach our TKGI cluster.

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-06-08-tanzu-mission-control-self-managed-installation/tmc-sm-attach-cluster.png" alt="Attach Cluster" title="Attach Cluster">
<img class="popup-img" src="/assets/images/2025-06-08-tanzu-mission-control-self-managed-installation/tmc-sm-attach-cluster-2.png" alt="Attach Cluster" title="Attach Cluster">
<img class="popup-img" src="/assets/images/2025-06-08-tanzu-mission-control-self-managed-installation/tmc-sm-attach-cluster-3.png" alt="Attach Cluster" title="Attach Cluster">
<img class="popup-img" src="/assets/images/2025-06-08-tanzu-mission-control-self-managed-installation/tmc-sm-attach-cluster-verify-connection.png" alt="Attach Cluster" title="Attach Cluster">
</div>
<figcaption>Attach a TKGI Cluster</figcaption>
</figure> 

Our TKGI cluster has been attached successfully!  We can see our Openldap workload app there as well.

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-06-08-tanzu-mission-control-self-managed-installation/tmc-sm-cluster-attached.png" alt="TKGI Cluster" title="TKGI Cluster">
<img class="popup-img" src="/assets/images/2025-06-08-tanzu-mission-control-self-managed-installation/tmc-sm-openldap-workload.png" alt="Openldap Workload" title="Openldap Workload">
</div>
<figcaption>TKGI Cluster and the Openldap workload</figcaption>
</figure> 

That concludes our task to install the VMware Tanzu Mission Control Self-Managed!
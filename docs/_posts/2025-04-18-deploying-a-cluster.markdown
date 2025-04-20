---
layout: post
title:  "Deploying a TKGI cluster"
date:   2025-04-18 19:53:00 -0400
categories: deephackmode.io update
---
In this post, I will create a small TKGI cluster, with a control-plane and a worker.  

I will use the Ops Manager VM as a jumpbox where I can run the commands to communicate with the Bosh Director, the TKGI API and the kubernetes API server.

To start with, I will log in to the Ops Manager VM via ssh.  I will use the private key that was the pair to the public key used during the [deployment of the Ops Manager OVA](/deephackmode.io/update/2025/04/16/deploying-a-tkgi-foundation.html#deploy-the-tanzu-operations-manager){:target="_blank"}.  This is the command I used to ssh to the Ops Manager VM.

{% include codeblock.html code="ssh -i ssh.key ubuntu@opsmgr.deephackmode.io" %}

Once logged in to the Ops Manager VM, I will create a file named `env.sh` that will set up environment variables needed to communicate with the Bosh Director.  Before I do that, I will get the Bosh Commandline Credentials from the Ops Manager.  This can be retrieved from Ops Manager->Bosh Director tile->Credentials->Bosh Commandline Credentials.  Add the `export` command at the beginning of the line when you enter it in the file.  An example of my `env.sh` file is:

e.g.,
```
$ cat env.sh
export BOSH_CLIENT=ops_manager BOSH_CLIENT_SECRET=xxx BOSH_CA_CERT=/var/tempest/workspaces/default/root_ca_certificate BOSH_ENVIRONMENT=10.1.1.11

```

To source it, just run it like:
{% include codeblock.html code=". env.sh" %}

Now that I have the necessary envars set, I can now run bosh commands.

e.g.,
```
$ bosh vms
Using environment '10.1.1.11' as client 'ops_manager'

Task 669
Task 669 done

Deployment 'pivotal-container-service-9db11bf32763a7a53575'

Instance                                                        Process State  AZ   IPs        VM CID                                   VM Type     Active  Stemcell
pivotal-container-service/67599633-9084-46d4-b9dd-4895f5b6a613  running        az1  10.1.1.13  vm-c931a2c6-0942-4637-a4ad-7bd68aa435f7  large.disk  true    bosh-vsphere-esxi-ubuntu-jammy-go_agent/1.785
pks-db/03025397-6bc2-4b80-9bc0-6ca531b843ab                     running        az1  10.1.1.12  vm-7e7817d8-b864-4cca-9ef1-fba842f165fe  large.disk  true    bosh-vsphere-esxi-ubuntu-jammy-go_agent/1.785

2 vms

Succeeded
```

Next, I will set up the tkgi cli to communicate with the TKGI API server.

Download the tkgi and kubectl cli binaries from the Broadcom Download Portal to the Ops Manager VM.  I will install them using the `install` command:

{% include codeblock.html code="sudo install tkgi-linux-amd64-1.21.0-build.55 /usr/local/bin/tkgi" %}

{% include codeblock.html code="sudo install kubectl-linux-amd64-1.30.7 /usr/local/bin/kubectl" %}

I will retrieve the "Uaa Admin Password" from Ops Manager->TKGI tile->Credentials->Uaa Admin Password.  I will add a line in my `env.sh` file to log in to the TKGI API server.  My updated `env.sh` looks like this now:

e.g.,
```
$ cat env.sh
export BOSH_CLIENT=ops_manager BOSH_CLIENT_SECRET=xxx BOSH_CA_CERT=/var/tempest/workspaces/default/root_ca_certificate BOSH_ENVIRONMENT=10.1.1.11
tkgi login -a tkgi.deephackmode.io -u admin -p xxx -k

```

I will run source `.env.sh` again and the output will look like this:

e.g.,
```
$ . env.sh

API Endpoint: tkgi.deephackmode.io
User: admin
Login successful.

```

I will now create a cluster by running the following command:
{% include codeblock.html code="tkgi create-cluster orion -p small -n 1 -e orion.deephackmode.io" %}

In the above example, `orion` is the cluster name, the plan is `small`, the number of workers is `1`, and the FQDN of the cluster's API server is `orion.deephackmode.io`.

Wait for it to complete, by either running `bosh task` or `tkgi cluster orion` to check on status.  

If cluster creation was successful, then the Last Action State would show 'succeeded'.

e.g.,
```
$ tkgi cluster orion

PKS Version:              1.21.0-build.32
Name:                     orion
K8s Version:              1.30.7
Plan Name:                small
UUID:                     6907a1e2-979e-4c6a-bc71-ce688837c178
Last Action:              CREATE
Last Action State:        succeeded
Last Action Description:  Instance update completed
Kubernetes Master Host:   orion.deephackmode.io
Kubernetes Master Port:   8443
Worker Nodes:             1
Kubernetes Master IP(s):  192.168.16.101
Network Profile Name:
Kubernetes Profile Name:
Compute Profile Name:
NSX Policy:               true
Private Registries:       false
Tags:
```

Set up DNS to add an entry for the "Kubernetes Master Host" and the "Kubernetes Master IP" seen in the output above.

Log in to the newly created cluster by running `get-credentials`.
{% include codeblock.html code="tkgi get-credentials orion" %}

You should now be able to examine the cluster using the `kubectl` cli.  

To see the nodes in the cluster, run:

{% include codeblock.html code="kubectl get nodes" %}

To see all the pods in the cluster, run:
{% include codeblock.html code="kubectl get pods -A" %}

e.g.,
```
$ kubectl get nodes
NAME                                   STATUS   ROLES    AGE   VERSION
41de4611-9d74-4a9f-bbbd-c12efd25cf3d   Ready    <none>   12m   v1.30.7+vmware.1

$ kubectl get pods -A
NAMESPACE           NAME                                                            READY   STATUS      RESTARTS   AGE
kube-system         coredns-85dd57f7df-zq6js                                        1/1     Running     0          10m
kube-system         metrics-server-7f95dbcbdf-msdv6                                 1/1     Running     0          10m
kube-system         snapshot-controller-59ffcb8bc5-bsjx2                            1/1     Running     0          10m
kube-system         snapshot-controller-59ffcb8bc5-pl425                            1/1     Running     0          10m
kube-system         snapshot-validation-deployment-6b5f854df8-8pz2l                 1/1     Running     0          10m
kube-system         snapshot-validation-deployment-6b5f854df8-jh5zv                 1/1     Running     0          10m
kube-system         snapshot-validation-deployment-6b5f854df8-vhz4c                 1/1     Running     0          10m
pks-system          cert-generator-582965efbcdf1520914a29ffaf018ef3e7bf5a19-frvxn   0/1     Completed   0          10m
pks-system          event-controller-fb457457d-4gcn9                                2/2     Running     0          10m
pks-system          fluent-bit-svbdx                                                2/2     Running     0          10m
pks-system          metric-controller-595c84d4c7-4ppzs                              1/1     Running     0          10m
pks-system          observability-manager-6b8c77c5f-vtnxb                           1/1     Running     0          10m
pks-system          sink-controller-7f67fb549c-d9r5x                                1/1     Running     0          10m
pks-system          telegraf-jr8wr                                                  1/1     Running     0          10m
pks-system          validator-6bdd686b8-q8q4m                                       1/1     Running     0          10m
vmware-system-csi   vsphere-csi-webhook-678bd7579-92t4w                             1/1     Running     0          10m
vmware-system-csi   vsphere-csi-webhook-678bd7579-wd2g9                             1/1     Running     0          10m
vmware-system-csi   vsphere-csi-webhook-678bd7579-wl2s8                             1/1     Running     0          10m


```

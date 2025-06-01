---
layout: post
title:  "Preparing the Workload Cluster for the TMC-SM installation"
date:   2025-06-01 11:37:00 -0400
categories: deephackmode.io update
---
Now that I have a [workload cluster up and running](https://deephackmode.io/deephackmode.io/update/2025/05/22/creating-a-tkgm-workload-cluster.html#create-the-tkgm-workload-cluster){:target="_blank"}, I then needed to prepare it for Tanzu Mission Control - Self Managed (TMC-SM) installation.

Before I go any further, I wanted to create a couple of command aliases for `kubectl` and `tanzu` commands.  Also, I wanted to enable command auto-completion on these as well.  I know I can type relatively fast but I am also lazy so I just wanted to shorten these commands for convenience.

### Set up command aliases and auto-completion

To accomplish command auto-completion, the bash-completion package needs to be installed.

{% include codeblock.html code="sudo apt update
sudo apt install bash-completion
" %}

Then, create a shell script with the following commands:
{% include codeblock.html code="cat >> env.sh <<EOF
. /usr/share/bash-completion/bash_completion
source <(kubectl completion bash|sed 's/kubectl/k/g')
alias k=kubectl
source <(tanzu completion bash|sed 's/tanzu/t/g')
alias t=tanzu
EOF
" %}

Source the `env.sh` file.
{% include codeblock.html code=". env.sh" %}

Now, you can run `t` for `tanzu` and `k` for kubectl.
```
ubuntu@numbat:~$ t cluster list
  NAME     NAMESPACE  STATUS   CONTROLPLANE  WORKERS  KUBERNETES        ROLES   PLAN  TKR
  av-wkld  default    running  3/3           3/3      v1.30.2+vmware.1  <none>  dev   v1.30.2---vmware.1-tkg.1
ubuntu@numbat:~$ k get no
NAME                               STATUS   ROLES           AGE   VERSION
av-mgmt-controlplane-8chpd-khb48   Ready    control-plane   19d   v1.28.11+vmware.2
av-mgmt-md-0-pfwtg-lj65v-fx8qm     Ready    <none>          19d   v1.28.11+vmware.2
ubuntu@numbat:~$
```

Also, auto-completion works in these 2 commands as well.

e.g.,
```
t cluster list -A --in
```

Typing the above, and then pressing the Tab key will auto-complete it for you to:
```
t cluster list -A --include-management-cluster
```

You get the point.  Imagine the convenience for auto-completing the word `vspheremachinetemplates`, for example.


Now back to preparing the cluster.  

Used the official [TMC-SM doc](https://techdocs.broadcom.com/us/en/vmware-tanzu/standalone-components/tanzu-mission-control/1-4/tanzu-mission-control-documentation/tanzumc-sm-install-config-prepare-cluster.html){:target="_blank"} as a guide to prepare the environment.

Based on the doc, for non-production set up, I could use a workload cluster that has 1 control-plane and 3 worker nodes.  Each node should have 4 CPUs, 8GB memory and 40GB disk.

So, I had to increase the number of nodes in my workload cluster.  The cluster was created with just 1 control-plane and 2 worker nodes. 

I also had to make sure that my nodes have the miniumum resources as per the recommended specs.

### Horizontal Scaling

My workload cluster `av-wkld` only has 1 control-plane and 2 worker nodes.

```
ubuntu@numbat:~$ t cluster list --include-management-cluster -A
  NAME     NAMESPACE   STATUS   CONTROLPLANE  WORKERS  KUBERNETES         ROLES       PLAN  TKR
  av-wkld  default     running  1/1           2/2      v1.30.2+vmware.1   <none>      dev   v1.30.2---vmware.1-tkg.1
  av-mgmt  tkg-system  running  1/1           1/1      v1.28.11+vmware.2  management  dev   v1.28.11---vmware.2-tkg.2
ubuntu@numbat:~$ k config use-context av-wkld-admin@av-wkld
Switched to context "av-wkld-admin@av-wkld".
ubuntu@numbat:~$ k get no -o wide
NAME                               STATUS   ROLES           AGE   VERSION            INTERNAL-IP      EXTERNAL-IP      OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
av-wkld-controlplane-7xjv9-l7bh8   Ready    control-plane   9d    v1.30.2+vmware.1   192.168.86.155   192.168.86.155   Ubuntu 22.04.4 LTS   5.15.0-116-generic   containerd://1.6.33+vmware.2
av-wkld-md-0-hctds-vn6gp-7j9g7     Ready    <none>          9d    v1.30.2+vmware.1   192.168.86.156   192.168.86.156   Ubuntu 22.04.4 LTS   5.15.0-116-generic   containerd://1.6.33+vmware.2
av-wkld-md-0-hctds-vn6gp-r7gwj     Ready    <none>          9d    v1.30.2+vmware.1   192.168.86.157   192.168.86.157   Ubuntu 22.04.4 LTS   5.15.0-116-generic   containerd://1.6.33+vmware.2
ubuntu@numbat:~$ 
```

To increase the number of control-plane nodes to 3, and the worker nodes to 3, I ran the following command:

{% include codeblock.html code="t cluster scale av-wkld -c 3 -w 3" %}

The output was:
```
ubuntu@numbat:~$ t cluster scale av-wkld -c 3 -w 3
Workload cluster 'av-wkld' is being scaled
ubuntu@numbat:~$
```

That started the creation of additional nodes.  To monitor progress, I ran the `cluster get` command:

```
ubuntu@numbat:~$ t cluster get av-wkld --show-details
  NAME     NAMESPACE  STATUS    CONTROLPLANE  WORKERS  KUBERNETES        ROLES   TKR
  av-wkld  default    updating  2/3           3/3      v1.30.2+vmware.1  <none>  v1.30.2---vmware.1-tkg.1


Details:

NAME                                                                           READY  SEVERITY  REASON     SINCE  MESSAGE
/av-wkld                                                                       False  Warning   ScalingUp  5m59s  Scaling up control plane to 3 replicas (actual 2)
├─ClusterInfrastructure - VSphereCluster/av-wkld-7k82l                         True                        9d
├─ControlPlane - KubeadmControlPlane/av-wkld-controlplane-7xjv9                False  Warning   ScalingUp  5m59s  Scaling up control plane to 3 replicas (actual 2)
│ ├─Machine/av-wkld-controlplane-7xjv9-6nk4v                                   True                        62s
│ │ ├─BootstrapConfig - KubeadmConfig/av-wkld-controlplane-7xjv9-6nk4v         True                        5m59s
│ │ └─MachineInfrastructure - VSphereMachine/av-wkld-controlplane-7xjv9-6nk4v  True                        62s
│ ├─Machine/av-wkld-controlplane-7xjv9-kf8q8                                   False  Info      Cloning    3s     1 of 2 completed
│ │ ├─BootstrapConfig - KubeadmConfig/av-wkld-controlplane-7xjv9-kf8q8         True                        3s
│ │ └─MachineInfrastructure - VSphereMachine/av-wkld-controlplane-7xjv9-kf8q8  False  Info      Cloning    3s
│ └─Machine/av-wkld-controlplane-7xjv9-l7bh8                                   True                        9d
│   ├─BootstrapConfig - KubeadmConfig/av-wkld-controlplane-7xjv9-l7bh8         True                        9d
│   └─MachineInfrastructure - VSphereMachine/av-wkld-controlplane-7xjv9-l7bh8  True                        9d
└─Workers
  └─MachineDeployment/av-wkld-md-0-hctds                                       True                        46s
    ├─Machine/av-wkld-md-0-hctds-vn6gp-7j9g7                                   True                        9d
    │ ├─BootstrapConfig - KubeadmConfig/av-wkld-md-0-hctds-vn6gp-7j9g7         True                        9d
    │ └─MachineInfrastructure - VSphereMachine/av-wkld-md-0-hctds-vn6gp-7j9g7  True                        9d
    ├─Machine/av-wkld-md-0-hctds-vn6gp-dkczf                                   True                        62s
    │ ├─BootstrapConfig - KubeadmConfig/av-wkld-md-0-hctds-vn6gp-dkczf         True                        5m59s
    │ └─MachineInfrastructure - VSphereMachine/av-wkld-md-0-hctds-vn6gp-dkczf  True                        62s
    └─Machine/av-wkld-md-0-hctds-vn6gp-r7gwj                                   True                        9d
      ├─BootstrapConfig - KubeadmConfig/av-wkld-md-0-hctds-vn6gp-r7gwj         True                        9d
      └─MachineInfrastructure - VSphereMachine/av-wkld-md-0-hctds-vn6gp-r7gwj  True                        9d
ubuntu@numbat:~$
```

Once completed, it shows the new numbers of the nodes and status is back to 'running'.
```
ubuntu@numbat:~$ t cluster get av-wkld --show-details
  NAME     NAMESPACE  STATUS   CONTROLPLANE  WORKERS  KUBERNETES        ROLES   TKR
  av-wkld  default    running  3/3           3/3      v1.30.2+vmware.1  <none>  v1.30.2---vmware.1-tkg.1


Details:

NAME                                                                           READY  SEVERITY  REASON  SINCE  MESSAGE
/av-wkld                                                                       True                     5s
├─ClusterInfrastructure - VSphereCluster/av-wkld-7k82l                         True                     9d
├─ControlPlane - KubeadmControlPlane/av-wkld-controlplane-7xjv9                True                     6s
│ ├─Machine/av-wkld-controlplane-7xjv9-6nk4v                                   True                     5m3s
│ │ ├─BootstrapConfig - KubeadmConfig/av-wkld-controlplane-7xjv9-6nk4v         True                     10m
│ │ └─MachineInfrastructure - VSphereMachine/av-wkld-controlplane-7xjv9-6nk4v  True                     5m3s
│ ├─Machine/av-wkld-controlplane-7xjv9-kf8q8                                   True                     11s
│ │ ├─BootstrapConfig - KubeadmConfig/av-wkld-controlplane-7xjv9-kf8q8         True                     4m4s
│ │ └─MachineInfrastructure - VSphereMachine/av-wkld-controlplane-7xjv9-kf8q8  True                     11s
│ └─Machine/av-wkld-controlplane-7xjv9-l7bh8                                   True                     9d
│   ├─BootstrapConfig - KubeadmConfig/av-wkld-controlplane-7xjv9-l7bh8         True                     9d
│   └─MachineInfrastructure - VSphereMachine/av-wkld-controlplane-7xjv9-l7bh8  True                     9d
└─Workers
  └─MachineDeployment/av-wkld-md-0-hctds                                       True                     4m47s
    ├─Machine/av-wkld-md-0-hctds-vn6gp-7j9g7                                   True                     9d
    │ ├─BootstrapConfig - KubeadmConfig/av-wkld-md-0-hctds-vn6gp-7j9g7         True                     9d
    │ └─MachineInfrastructure - VSphereMachine/av-wkld-md-0-hctds-vn6gp-7j9g7  True                     9d
    ├─Machine/av-wkld-md-0-hctds-vn6gp-dkczf                                   True                     5m3s
    │ ├─BootstrapConfig - KubeadmConfig/av-wkld-md-0-hctds-vn6gp-dkczf         True                     10m
    │ └─MachineInfrastructure - VSphereMachine/av-wkld-md-0-hctds-vn6gp-dkczf  True                     5m3s
    └─Machine/av-wkld-md-0-hctds-vn6gp-r7gwj                                   True                     9d
      ├─BootstrapConfig - KubeadmConfig/av-wkld-md-0-hctds-vn6gp-r7gwj         True                     9d
      └─MachineInfrastructure - VSphereMachine/av-wkld-md-0-hctds-vn6gp-r7gwj  True                     9d
ubuntu@numbat:~$
```

### Vertical Scaling

I realized that my nodes only have 2 CPUs each.  They have 8GB memory and 40GB storage, which are aligned to the recommended specs.

I used the TKGm doc on [how to scale the class-based cluster vertically](https://techdocs.broadcom.com/us/en/vmware-tanzu/standalone-components/tanzu-kubernetes-grid/2-5/tkg/workload-clusters-scale.html#vertical:~:text=Based%20Cluster%20below.-,Scale%20a%20Class%2DBased%20Cluster,-To%20scale%20a){:target="_blank"}.

In the management cluster context, I edited the cluster 'av-wkld' object.

{% include codeblock.html code="k edit cluster av-wkld" %}

I changed the `numCPUs` from `2` to `4`.

e.g.,
```
    - name: controlPlane
      value:
        machine:
          customVMXKeys: {}
          diskGiB: 40
          memoryMiB: 8192
          numCPUs: 4
          numCoresPerSocket: 0
        network:
          nameservers:
          - 192.168.86.34
          searchDomains: []
        nodeLabels: []
    - name: worker
      value:
        machine:
          customVMXKeys: {}
          diskGiB: 40
          memoryMiB: 8192
          numCPUs: 4
          numCoresPerSocket: 0
        network:
          nameservers:
          - 192.168.86.34
          searchDomains: []

```
Saved the changes and quit the editor.

As soon as I quit the editor, I checked the cluster status and it's showing `updating` already.

```
ubuntu@numbat:~$ t cluster list
  NAME     NAMESPACE  STATUS    CONTROLPLANE  WORKERS  KUBERNETES        ROLES   PLAN  TKR
  av-wkld  default    updating  3/3           3/3      v1.30.2+vmware.1  <none>  dev   v1.30.2---vmware.1-tkg.1
ubuntu@numbat:~$
```

I also monitored the progress from the vCenter.  At some point, I saw that new machines are being cloned.

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-06-01-preparing-the-workload-cluster/vcenter-vm-cloning.png" alt="VM cloning" title="VM cloning">
<img class="popup-img" src="/assets/images/2025-06-01-preparing-the-workload-cluster/new-vms-with-4cpus.png" alt="VM's with 4 CPUs" title="VM's with 4 CPUs">
</div>
<figcaption>vCenter shows VM's being cloned and now have 4 CPUs</figcaption>
</figure> 

After a few minutes, the vertical scaling completed.  The cluster status is now back to `running`.

```
ubuntu@numbat:~$ t cluster list
  NAME     NAMESPACE  STATUS   CONTROLPLANE  WORKERS  KUBERNETES        ROLES   PLAN  TKR
  av-wkld  default    running  3/3           3/3      v1.30.2+vmware.1  <none>  dev   v1.30.2---vmware.1-tkg.1
ubuntu@numbat:~$
```

At this point, the workload cluster is now ready for TMC-SM installation.

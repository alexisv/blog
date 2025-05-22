---
layout: post
title:  "Creating a TKGM Workload Cluster"
date:   2025-05-21 23:06:00 -0400
categories: deephackmode.io update
---
This time I created a Workload Cluster using the Ubuntu image with k8s version 1.30.2.

### Download the TKGM Ubuntu OVA

I downloaded the Ubuntu 2204 Kubernetes v1.30.2 OVA image from the Broadcom Support Portal.

### Upload the image to vCenter

I uploaded the image to vCenter using the [same steps](http://127.0.0.1:4000/deephackmode.io/update/2025/05/13/creating-management-cluster.html#upload-the-base-image){:target="_blank"} that I ran during Management Cluster creation.


### Create a Cluster IP Pool

I wanted to create an IP pool for the cluster nodes so that I can control what IP addresses are assigned to them.

```
ubuntu@numbat:~$ cat workload-GlobalInClusterIPPool.yaml
apiVersion: ipam.cluster.x-k8s.io/v1alpha2
kind: GlobalInClusterIPPool
metadata:
  name: workload-globalincluster-ippool
spec:
  gateway: 192.168.86.34
  addresses:
  - 192.168.86.155-192.168.86.167
  prefix: 24
ubuntu@numbat:~$ kubectl apply -f workload-GlobalInClusterIPPool.yaml
globalinclusterippool.ipam.cluster.x-k8s.io/workload-globalincluster-ippool created
ubuntu@numbat:~$

```

### Create the Workload Cluster configuration file

I copied the Management Cluster config file and changed a few settings such as CLUSTER_NAME and VSPHERE_TEMPLATE.  I also removed all of the settings with names that start with "MANAGEMENT_NODE_IPAM_IP_POOL", and replaced them with the NODE_IPAM_IP_POOL_* settings.

```
$ cat av-wkld.yaml
AVI_ENABLE: true
AVI_CA_DATA_B64: LS0XXXX....XXXXXXXX=
AVI_CLOUD_NAME: "vmware-cloud"
#AVI_CONTROLLER: "avi-controller.deephackmode.io"
AVI_CONTROLLER: "192.168.86.211"
AVI_CONTROL_PLANE_HA_PROVIDER: true
AVI_DATA_NETWORK: "VM Network"
AVI_DATA_NETWORK_CIDR: 192.168.86.0/24
AVI_USERNAME: "admin"
AVI_PASSWORD: 'xxx'
AVI_SERVICE_ENGINE_GROUP: "Default-Group"
CLUSTER_CIDR: 100.96.0.0/11
CLUSTER_NAME: av-wkld
CLUSTER_PLAN: dev
ENABLE_TKGS_ON_VSPHERE7: "false"
DEPLOY_TKG_ON_VSPHERE7: "true"
ENABLE_CEIP_PARTICIPATION: "false"
ENABLE_MHC: "true"
IDENTITY_MANAGEMENT_TYPE: none
INFRASTRUCTURE_PROVIDER: vsphere
SERVICE_CIDR: 100.64.0.0/13
TKG_HTTP_PROXY_ENABLED: "false"
VSPHERE_CONTROL_PLANE_DISK_GIB: "40"
VSPHERE_CONTROL_PLANE_MEM_MIB: "8192"
VSPHERE_CONTROL_PLANE_NUM_CPUS: "2"
VSPHERE_DATACENTER: /Datacenter
VSPHERE_DATASTORE: /Datacenter/datastore/datastore1
VSPHERE_FOLDER: /Datacenter/vm/tkgm
VSPHERE_NETWORK: VM Network
VSPHERE_USERNAME: administrator@vsphere.local
VSPHERE_PASSWORD: <encoded:xxx>
VSPHERE_RESOURCE_POOL: TKGM
VSPHERE_SERVER: vc-1.deephackmode.io
VSPHERE_SSH_AUTHORIZED_KEY: ssh-rsa XXXXXXXX...XXXXXX== avillalon@vmware.com
VSPHERE_TLS_THUMBPRINT: XX:4F:97:88:14:73:74:D0:21:0F:DE:A7:18:7B:8E:FF
VSPHERE_WORKER_DISK_GIB: "40"
VSPHERE_WORKER_MEM_MIB: "8192"
VSPHERE_WORKER_NUM_CPUS: "2"
VSPHERE_TEMPLATE: /Datacenter/vm/tkgm/ubuntu-2204-efi-kube-v1.30.2
NODE_IPAM_IP_POOL_KIND: "GlobalInClusterIPPool"
NODE_IPAM_IP_POOL_NAME: "workload-globalincluster-ippool"
CONTROL_PLANE_NODE_NAMESERVERS: "192.168.86.34"
WORKER_NODE_NAMESERVERS: "192.168.86.34"
WORKER_MACHINE_COUNT: "2"
TKG_CUSTOM_IMAGE_REPOSITORY: "harbor.deephackmode.io/tkg"
TKG_CUSTOM_IMAGE_REPOSITORY_CA_CERTIFICATE: "LS0XXXXXXXX...XXXX="

```


### Create the TKGM Workload Cluster

I needed to get the TKR (Tanzu Kubernetes Release) for the version that I uploaded in the initial step.  That was v1.30.2.

```
ubuntu@numbat:~$ tanzu kubernetes-release get
  NAME                              VERSION                         COMPATIBLE  ACTIVE  UPDATES AVAILABLE
  v1.26.14---vmware.1-tiny.1-tkg.3  v1.26.14+vmware.1-tiny.1-tkg.3  True        True
  v1.26.14---vmware.1-tkg.4         v1.26.14+vmware.1-tkg.4         True        True
  v1.27.13---vmware.1-tiny.1-tkg.2  v1.27.13+vmware.1-tiny.1-tkg.2  True        True
  v1.27.15---vmware.1-tkg.2         v1.27.15+vmware.1-tkg.2         True        True
  v1.28.11---vmware.2-tkg.2         v1.28.11+vmware.2-tkg.2         True        True
  v1.28.9---vmware.1-tiny.1-tkg.2   v1.28.9+vmware.1-tiny.1-tkg.2   True        True
  v1.29.6---vmware.1-tkg.3          v1.29.6+vmware.1-tkg.3          True        True
  v1.30.2---vmware.1-tkg.1          v1.30.2+vmware.1-tkg.1          True        True

```
I ran the following command to start the cluster creation.  I specified the tkr I got from the above output.  If I don't specify the TKR, it will go looking for an Ubuntu image version 1.28.11 by default and it will fail in this case because I didn't upload that particular image.

```
tanzu cluster create -f av-wkld.yaml -v 6 --tkr v1.30.2---vmware.1-tkg.1

```

It completed quickly but only because it only generated another cluster config file.  See the output below.

```
...
Legacy configuration file detected. The inputs from said file have been converted into the new Cluster configuration as '/home/ubuntu/.config/tanzu/tkg/clusterconfigs/av-wkld.yaml'

To create a cluster with it, please use the following command together with all customized flags set in the last step. e.g. --tkr, -v
    tanzu cluster create --file /home/ubuntu/.config/tanzu/tkg/clusterconfigs/av-wkld.yaml

```

I then ran the following command to start the actual cluster creation:

```
tanzu cluster create --file /home/ubuntu/.config/tanzu/tkg/clusterconfigs/av-wkld.yaml -v 6
```


After a few minutes, it has completed successfully.  

```

Workload cluster 'av-wkld' created

```

Basic checks of the workload cluster, pods and packages were all fine.

```
ubuntu@numbat:~$ tanzu cluster list --include-management-cluster -A
  NAME     NAMESPACE   STATUS   CONTROLPLANE  WORKERS  KUBERNETES         ROLES       PLAN  TKR
  av-wkld  default     running  1/1           2/2      v1.30.2+vmware.1   <none>      dev   v1.30.2---vmware.1-tkg.1
  av-mgmt  tkg-system  running  1/1           1/1      v1.28.11+vmware.2  management  dev   v1.28.11---vmware.2-tkg.2
ubuntu@numbat:~$ tanzu cluster get av-wkld --show-details
  NAME     NAMESPACE  STATUS   CONTROLPLANE  WORKERS  KUBERNETES        ROLES   TKR
  av-wkld  default    running  1/1           2/2      v1.30.2+vmware.1  <none>  v1.30.2---vmware.1-tkg.1


Details:

NAME                                                                           READY  SEVERITY  REASON  SINCE  MESSAGE
/av-wkld                                                                       True                     8m25s
├─ClusterInfrastructure - VSphereCluster/av-wkld-7k82l                         True                     12m
├─ControlPlane - KubeadmControlPlane/av-wkld-controlplane-7xjv9                True                     8m25s
│ └─Machine/av-wkld-controlplane-7xjv9-l7bh8                                   True                     8m46s
│   ├─BootstrapConfig - KubeadmConfig/av-wkld-controlplane-7xjv9-l7bh8         True                     12m
│   └─MachineInfrastructure - VSphereMachine/av-wkld-controlplane-7xjv9-l7bh8  True                     8m46s
└─Workers
  └─MachineDeployment/av-wkld-md-0-hctds                                       True                     2m9s
    ├─Machine/av-wkld-md-0-hctds-vn6gp-7j9g7                                   True                     2m12s
    │ ├─BootstrapConfig - KubeadmConfig/av-wkld-md-0-hctds-vn6gp-7j9g7         True                     7m23s
    │ └─MachineInfrastructure - VSphereMachine/av-wkld-md-0-hctds-vn6gp-7j9g7  True                     2m12s
    └─Machine/av-wkld-md-0-hctds-vn6gp-r7gwj                                   True                     2m22s
      ├─BootstrapConfig - KubeadmConfig/av-wkld-md-0-hctds-vn6gp-r7gwj         True                     7m23s
      └─MachineInfrastructure - VSphereMachine/av-wkld-md-0-hctds-vn6gp-r7gwj  True                     2m22s
ubuntu@numbat:~$
```

```
ubuntu@numbat:~$ tanzu cluster kubeconfig get av-wkld --admin
Credentials of cluster 'av-wkld' have been saved
You can now access the cluster by running 'kubectl config use-context av-wkld-admin@av-wkld'
ubuntu@numbat:~$ kubectl config use-context av-wkld-admin@av-wkld
Switched to context "av-wkld-admin@av-wkld".
ubuntu@numbat:~$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.86.207:6443
CoreDNS is running at https://192.168.86.207:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
ubuntu@numbat:~$


ubuntu@numbat:~$ kubectl get no -o wide
NAME                               STATUS   ROLES           AGE     VERSION            INTERNAL-IP      EXTERNAL-IP      OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
av-wkld-controlplane-7xjv9-l7bh8   Ready    control-plane   11m     v1.30.2+vmware.1   192.168.86.155   192.168.86.155   Ubuntu 22.04.4 LTS   5.15.0-116-generic   containerd://1.6.33+vmware.2
av-wkld-md-0-hctds-vn6gp-7j9g7     Ready    <none>          5m10s   v1.30.2+vmware.1   192.168.86.156   192.168.86.156   Ubuntu 22.04.4 LTS   5.15.0-116-generic   containerd://1.6.33+vmware.2
av-wkld-md-0-hctds-vn6gp-r7gwj     Ready    <none>          5m12s   v1.30.2+vmware.1   192.168.86.157   192.168.86.157   Ubuntu 22.04.4 LTS   5.15.0-116-generic   containerd://1.6.33+vmware.2
ubuntu@numbat:~$

ubuntu@numbat:~$ kubectl get po -A
NAMESPACE              NAME                                                       READY   STATUS      RESTARTS        AGE
avi-system             ako-0                                                      1/1     Running     0               10m
kube-system            antrea-agent-j6bmj                                         2/2     Running     0               10m
kube-system            antrea-agent-tzrfx                                         2/2     Running     0               5m44s
kube-system            antrea-agent-vsrzz                                         2/2     Running     0               5m42s
kube-system            antrea-controller-564d5bf96-7cng7                          1/1     Running     0               10m
kube-system            coredns-6b58858c85-cs7gr                                   1/1     Running     0               11m
kube-system            coredns-6b58858c85-dknfp                                   1/1     Running     0               11m
kube-system            etcd-av-wkld-controlplane-7xjv9-l7bh8                      1/1     Running     0               11m
kube-system            kube-apiserver-av-wkld-controlplane-7xjv9-l7bh8            1/1     Running     0               11m
kube-system            kube-controller-manager-av-wkld-controlplane-7xjv9-l7bh8   1/1     Running     0               11m
kube-system            kube-proxy-7phqn                                           1/1     Running     0               5m42s
kube-system            kube-proxy-nz98b                                           1/1     Running     0               5m44s
kube-system            kube-proxy-vxk4w                                           1/1     Running     0               11m
kube-system            kube-scheduler-av-wkld-controlplane-7xjv9-l7bh8            1/1     Running     0               11m
kube-system            metrics-server-54566c779f-nhpcx                            1/1     Running     0               10m
kube-system            vsphere-cloud-controller-manager-57ttd                     1/1     Running     0               11m
secretgen-controller   secretgen-controller-54ff485688-lxnhq                      1/1     Running     0               10m
tkg-system             kapp-controller-645fc89db6-gl2lt                           2/2     Running     0               11m
tkg-system             tanzu-capabilities-controller-manager-85d7d9bcd-flmk5      1/1     Running     0               10m
vmware-system-antrea   register-placeholder-7sxg7                                 0/1     Completed   0               10m
vmware-system-csi      vsphere-csi-controller-5cdf9477cc-dh7qd                    7/7     Running     1 (9m49s ago)   11m
vmware-system-csi      vsphere-csi-node-t9r2k                                     3/3     Running     4 (9m41s ago)   11m
vmware-system-csi      vsphere-csi-node-w28dn                                     3/3     Running     0               5m44s
vmware-system-csi      vsphere-csi-node-x4qgd                                     3/3     Running     2 (5m1s ago)    5m42s
ubuntu@numbat:~$

ubuntu@numbat:~$ tanzu package installed list -A

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
ubuntu@numbat:~$
```


### Some clean up and Avi checks
In Avi, I deleted the "dummy" Virtual Service, which was [created](https://deephackmode.io/deephackmode.io/update/2025/05/09/installing-avi.html#create-a-dummy-virtual-service){:target="_blank"} as a test during the deployment of the Avi Controller.

I looked at the Applications and Infrastrucutre pages in the Avi Controller UI as well, and they appear to be fine.

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-05-21-creating-a-tkgm-workload-cluster/avi-apps-dash.png" alt="Avi Apps Dashboard" title="Avi Apps Dashboard">
<img class="popup-img" src="/assets/images/2025-05-21-creating-a-tkgm-workload-cluster/avi-apps-pools.png" alt="Avi Apps Pools" title="Avi Apps Pools">
<img class="popup-img" src="/assets/images/2025-05-21-creating-a-tkgm-workload-cluster/avi-apps-vs.png" alt="Avi Apps Virtual Servers" title="Avi Apps Virtual Servers">
<img class="popup-img" src="/assets/images/2025-05-21-creating-a-tkgm-workload-cluster/avi-apps-vs-vips.png" alt="Avi Apps Virtual Servers VIPs" title="Avi Apps Virtual Servers VIPs">
</div>
<figcaption>The Virtual Servers of the Management and Workload Clusters now show up in the Applications tabs in Avi Controller UI</figcaption>
</figure> 


<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-05-21-creating-a-tkgm-workload-cluster/avi-infra-dash.png" alt="Avi Infra Dashboard" title="Avi Infra Dashboard">
<img class="popup-img" src="/assets/images/2025-05-21-creating-a-tkgm-workload-cluster/avi-infra-se.png" alt="Avi Infra Service Engine" title="Avi Infra Service Engine">
<img class="popup-img" src="/assets/images/2025-05-21-creating-a-tkgm-workload-cluster/avi-infra-se-group.png" alt="Avi Infra Service Engine Group" title="Avi Infra Service Engine Group">
</div>
<figcaption>The Service Engines status look good in Avi Controller UI</figcaption>
</figure> 







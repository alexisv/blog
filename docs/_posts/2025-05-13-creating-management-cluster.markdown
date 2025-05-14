---
layout: post
title:  "Creating a Management Cluster"
date:   2025-05-13 23:17:00 -0400
categories: deephackmode.io update
---
It's time to create a TKGM Management Cluster!

### Download the TKGM Base Image

I downloaded the Photon v1.28.11 image from the Broadcom Support Portal.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-05-13-creating-management-cluster/tkgm-download-page.png" alt="TKGM v2.5.2 Download Page" title="TKGM v2.5.2 Download Page">
<img class="popup-img" src="/assets/images/2025-05-13-creating-management-cluster/tkgm-download-base-image.png" alt="TKGM v2.5.2 Base Image" title="TKGM v2.5.2 Base Image">
</div>
<figcaption>Download the TKGM Base Image</figcaption>
</figure> 

### Create a Resource Pool and Templates Folder in vCenter

I created a Resource Pool named "TKGM".  I also created a VM/Templates Folder and named it "tkgm".  These are to get these objects organized in vCenter.

### Upload the Base Image

I used the `govc` cli to upload the image and make it as a VM template.  I sourced the `govcenv.sh` file that I created in the previous post abount "Noble Numbat".  By doing this, I set up the required envars for `govc` to work with communicating to vCenter.

```
. govcenv.sh
```


I created a `import.spec` file based on the OVA file.  Here's the command:

```
govc import.spec photon-5-kube-v1.28.11+vmware.2-tkg.2-bc1be576772547364c88bf37cd99c838.ova | python3 -m json.tool > photon.json
```

I edited the JSON file, basically changing the Network info only.

```
$ cat photon.json 
{
    "DiskProvisioning": "flat",
    "IPAllocationPolicy": "dhcpPolicy",
    "IPProtocol": "IPv4",
    "NetworkMapping": [
        {
            "Name": "nic0",
            "Network": "VM Network"
        }
    ],
    "Annotation": "Cluster API vSphere image - VMware Photon OS 64-bit and Kubernetes v1.28.11+vmware.2 - https://github.com/kubernetes-sigs/cluster-api-provider-vsphere",
    "MarkAsTemplate": false,
    "PowerOn": false,
    "InjectOvfEnv": false,
    "WaitForIP": false,
    "Name": null
}
```

I then uploaded the OVA:
```
govc import.ova -options photon.json -folder /Datacenter/vm/tkgm photon-5-kube-v1.28.11+vmware.2-tkg.2-bc1be576772547364c88bf37cd99c838.ova
```

I verified the OVA has been uploaded:
```
$ govc ls /Datacenter/vm/tkgm
/Datacenter/vm/tkgm/photon-5-kube-v1.28.11
```

I then marked it as a template:
```
govc vm.markastemplate /Datacenter/vm/tkgm/photon-5-kube-v1.28.11
```


### Set up the tanzu cli

I installed `tanzu` cli in the previous post (Noble Numbat).  I ran the following commands to set up the cli and install the plugins.

```
tanzu init
tanzu plugin list
tanzu plugin sync
tanzu plugin group get vmware-tkg/default:v2.5.2
tanzu plugin install --group vmware-tkg/default:v2.5.2
tanzu plugin list
```

### Get the vCenter thumbprint info

I ran the following to get the vCenter thumbprint that will be used in the cluster config file.

```
$ echo | openssl s_client -connect vc-1.deephackmode.io:443 2>/dev/null | openssl x509 -noout -fingerprint -sha1
sha1 Fingerprint=2C:8D:09:73:32:4F:97:88:14:73:74:D0:21:0F:DE:A7:18:7B:8E:FF
```


### Get the encoded string of the vCenter password

This encoded string is also needed to be set in the cluster config file:

```
$  echo -n 'password' | base64
cGFzc3dvcmQ=
```


### Add a 100GB disk to numbat

I had to add a 100GB disk to my jumpbox because I will use an air-gapped installation method, which means that I need to transfer the container images from the public Broadcom registry to my private Harbor registry.

I shut down numbat.  Then, in vCenter, I edited the VM settings and then a new Hard Disk device (100GB in size).  I also increased the memory from 1 to 8GB.  I then powered it back on!

Once it's back up and running.  I ran the following commands to partition, format and mount the disk.

```
ubuntu@numbat:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
fd0       2:0    1    4K  0 disk
sda       8:0    0   10G  0 disk
├─sda1    8:1    0    9G  0 part /
├─sda14   8:14   0    4M  0 part
├─sda15   8:15   0  106M  0 part /boot/efi
└─sda16 259:0    0  913M  0 part /boot
sdb       8:16   0  100G  0 disk
sr0      11:0    1   44K  0 rom
ubuntu@numbat:~$ sudo -i
root@numbat:~# parted
GNU Parted 3.6
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) select /dev/sdb
Using /dev/sdb
(parted) mklabel gpt
(parted) print
Model: VMware Virtual disk (scsi)
Disk /dev/sdb: 107GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start  End  Size  File system  Name  Flags

(parted) mkpart primary ext4 0% 100%
(parted) print
Model: VMware Virtual disk (scsi)
Disk /dev/sdb: 107GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End    Size   File system  Name     Flags
 1      1049kB  107GB  107GB  ext4         primary

(parted) quit
Information: You may need to update /etc/fstab.

root@numbat:~# lsblk -f
NAME    FSTYPE  FSVER LABEL           UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
fd0
sda
├─sda1  ext4    1.0   cloudimg-rootfs b6729b3d-0cb7-4166-8709-bce594ecd93b    4.7G    46% /
├─sda14
├─sda15 vfat    FAT32 UEFI            DADD-8B1D                              98.2M     6% /boot/efi
└─sda16 ext4    1.0   BOOT            5b84c2cf-4483-451c-8408-d76d6e18d65d  707.1M    13% /boot
sdb
└─sdb1
sr0     iso9660       OVF ENV         2025-05-11-23-26-47-0
root@numbat:~# lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
fd0       2:0    1    4K  0 disk
sda       8:0    0   10G  0 disk
├─sda1    8:1    0    9G  0 part /
├─sda14   8:14   0    4M  0 part
├─sda15   8:15   0  106M  0 part /boot/efi
└─sda16 259:0    0  913M  0 part /boot
sdb       8:16   0  100G  0 disk
└─sdb1    8:17   0  100G  0 part
sr0      11:0    1   44K  0 rom
root@numbat:~# mkfs -t ext4 /dev/sdb1
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 26213888 4k blocks and 6553600 inodes
Filesystem UUID: fbd89617-bddc-4a40-a986-4af21c7649a0
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done
Writing inode tables: done
Creating journal (131072 blocks): done
Writing superblocks and filesystem accounting information: done

root@numbat:~# lsblk -f
NAME    FSTYPE  FSVER LABEL           UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
fd0
sda
├─sda1  ext4    1.0   cloudimg-rootfs b6729b3d-0cb7-4166-8709-bce594ecd93b    4.7G    46% /
├─sda14
├─sda15 vfat    FAT32 UEFI            DADD-8B1D                              98.2M     6% /boot/efi
└─sda16 ext4    1.0   BOOT            5b84c2cf-4483-451c-8408-d76d6e18d65d  707.1M    13% /boot
sdb
└─sdb1  ext4    1.0                   fbd89617-bddc-4a40-a986-4af21c7649a0
sr0     iso9660       OVF ENV         2025-05-11-23-26-47-0
root@numbat:~# mount /dev/sdb1 /mnt
root@numbat:~# cd /mnt
root@numbat:/mnt# df -h .
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1        98G   24K   93G   1% /mnt
root@numbat:/mnt# ls -l
total 16
drwx------ 2 root root 16384 May 11 23:29 lost+found
root@numbat:/mnt#
```


### Set up the docker env

The private Harbor Registry is the one that I installed in a recent post (Deploying Harbor Registry).

I had to install the Ops Manager CA certificate in numbat so that I can log in to the registry and be able to push images there.

```
root@numbat:/usr/local/share/ca-certificates# vi opsman-ca.crt
root@numbat:/usr/local/share/ca-certificates# update-ca-certificates
Updating certificates in /etc/ssl/certs...
rehash: warning: skipping ca-certificates.crt,it does not contain exactly one certificate or CRL
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
root@numbat:/usr/local/share/ca-certificates# systemctl restart docker
exit
logout

ubuntu@numbat:/mnt/ubuntu$ docker login harbor.deephackmode.io
Username: admin
Password:
WARNING! Your password will be stored unencrypted in /home/ubuntu/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
ubuntu@numbat:/mnt/ubuntu$
```

### Transfer images to the private registry

I followed the TKGM [doc on preparing for offline installation](https://techdocs.broadcom.com/us/en/vmware-tanzu/standalone-components/tanzu-kubernetes-grid/2-5/tkg/mgmt-reqs-prep-offline.html){:target="_blank"}, but basically these are the 3 steps on how to transfer the images.

```
tanzu isolated-cluster list-bundle --source-repo projects.registry.vmware.com/tkg --tkg-version v2.5.2

tanzu isolated-cluster download-bundle --source-repo projects.registry.vmware.com/tkg --tkg-version v2.5.2

tanzu isolated-cluster upload-bundle --source-directory ./ --destination-repo harbor.deephackmode.io/tkg --ca-certificate /usr/local/share/ca-certificates/opsman-ca.crt

```


### Added the certificate data

Followed this tanzu cli [doc on how to add the private registry CA cert to the tanzu config](https://techdocs.broadcom.com/us/en/vmware-tanzu/cli/tanzu-cli/1-3/cli/index.html#registry-certificate:~:text=To%20add%20the%20registry%20CA%20certificate%20to%20the%20Tanzu%20CLI){:target="_blank"}:


```
tanzu config cert add --host harbor.deephackmode.io --ca-certificate /usr/local/share/ca-certificates/opsman-ca.crt
```


### Set up Avi Controller Custom Certificate

I followed this Avi [doc on how to set up the Custom Certificate](https://techdocs.broadcom.com/us/en/vmware-tanzu/standalone-components/tanzu-kubernetes-grid/2-5/tkg/mgmt-reqs-network-nsx-alb-install.html#:~:text=Avi%20Controller%20Setup%3A%20Custom%20Certificate){:target="_blank"}

I downloaded the certificate PEM data and saved it into a file, and also got a base64-encoded string from it, because it is needed in the cluster config for creating the management cluster.


### Create the Management Cluster configuration file

```
$ cat av-mgmt.yaml
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
CLUSTER_NAME: av-mgmt
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
VSPHERE_TEMPLATE: /Datacenter/vm/tkgm/photon-5-kube-v1.28.11
MANAGEMENT_NODE_IPAM_IP_POOL_GATEWAY: "192.168.86.34"
MANAGEMENT_NODE_IPAM_IP_POOL_ADDRESSES: "192.168.86.150-192.168.86.155"
MANAGEMENT_NODE_IPAM_IP_POOL_SUBNET_PREFIX: "24"
CONTROL_PLANE_NODE_NAMESERVERS: "192.168.86.34"
WORKER_NODE_NAMESERVERS: "192.168.86.34"
OS_NAME: "photon"
OS_VERSION: "5"
OS_ARCH: "amd64"
TKG_CUSTOM_IMAGE_REPOSITORY: "harbor.deephackmode.io/tkg"
TKG_CUSTOM_IMAGE_REPOSITORY_CA_CERTIFICATE: "LS0XXXXXXXX...XXXX="

```


### Create the TKGM Management Cluster

I ran the following command to start the cluster creation.

```
tanzu management-cluster create -f mgmt.yaml -v 6
```





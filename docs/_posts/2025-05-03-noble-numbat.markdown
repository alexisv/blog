---
layout: post
title:  "Noble Numbat"
date:   2025-05-03 12:44:00 -0400
categories: deephackmode.io update
---
Noble Numbat.  Interesting name.  That's the codename of the latest major version of the Ubuntu Server that has Long Term Support (LTS).  That is Ubuntu Server 24.04 LTS (Noble Numbat).  I gather that the first word of the codename will be used to call the release, and so I'll call this "Noble".

In this post, I will create a "jumpbox" VM using the Noble cloud image.  I will use this jumpbox to run "tanzu" cli commands, primarily to create and manage TKGM management and workload clusters.

Let's get going.

### Download the Ubuntu image

Ubuntu images can be dowloaded from the [Ubuntu Cloud Images site](https://cloud-images.ubuntu.com/){:target="_blank"}. From that page, I selected the "noble" folder, and then selected the latest daily build date, which at this time was "20250430".  I found the file with the description of "VMware/Virtualbox OVA".  I downloaded that file.  It was 555MB in size.  The file name was `noble-server-cloudimg-amd64.ova`.

### Deploy the image to vSphere

1. In vCenter, right-click on the cluster object and then select "Deploy OVF Template".
1. In the "Select an OVF template" tab, select "Local file" and then upload the downloaded OVA file.  Then, click "Next".
1. In the "Select a name and folder" tab, leave the VM name as it is, which is `ubuntu-noble-24.04-cloudimg`.  Select the folder `pcf_vms` as the location.  Then, click "Next"
1. In the "Select a compute resource" tab, select the cluster and then click "Next".
1. In the "Review details" tab, just click "Next".
1. In the "Select storage" tab, click the Datastore, and then set the virtual disk format to "Thin Provision".  Then, click "Next".
1. In the "Select networks" tab, set VM Network to "VM Network".  Then, click "Next".
1. In the "Customize template" tab, set the hostname to "numbat".  Then, set the ssh public key, and default user password.  Then, click "Next".
1. In the "Ready to complete" tab, click "Finish".
1. Monitor the task in vCenter.  It should complete in less than 2 minutes.
1. Power on the VM.
1. Since DHCP is running in the network, the VM should have automatically got an IP address.
1. ssh into the VM using your private ssh key.  It will prompt to change the password.  Enter the password specified during the deployment, and then enter new passwords.  It will log you out.  
    <figure>
    <div class="image-row-big">
    <img class="popup-img" src="/assets/images/2025-05-03-noble-numbat/change-password.png" alt="Change password" title="Change password">
    </div>
    <figcaption>Change your password immediately.</figcaption>
    </figure> 
1. ssh into the VM again.  The VM should then be ready for use.
    <figure>
    <div class="image-row-big">
    <img class="popup-img" src="/assets/images/2025-05-03-noble-numbat/logged-in.png" alt="VM ready" title="VM ready">
    </div>
    <figcaption>Noble Numbat is now ready.</figcaption>
    </figure> 

    ```
    $ cat /etc/os-release
    PRETTY_NAME="Ubuntu 24.04.2 LTS"
    NAME="Ubuntu"
    VERSION_ID="24.04"
    VERSION="24.04.2 LTS (Noble Numbat)"
    VERSION_CODENAME=noble
    ID=ubuntu
    ID_LIKE=debian
    HOME_URL="https://www.ubuntu.com/"
    SUPPORT_URL="https://help.ubuntu.com/"
    BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
    PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
    UBUNTU_CODENAME=noble
    LOGO=ubuntu-logo

    $ uname -a
    Linux noble 6.8.0-58-generic #60-Ubuntu SMP PREEMPT_DYNAMIC Fri Mar 14 18:29:48 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
    ```


### Configure the networking settings

I'll disable DHCP, and configure the DNS settings, IP address and default gateway.

Edit /etc/netplan/50-cloud-init.yaml:
```
network:
  version: 2
  ethernets:
    ens192:
      nameservers:
        search: [deephackmode.io]
        addresses: [192.168.86.34]
      match:
        macaddress: "00:50:56:b2:3b:cc"
      dhcp4: false
      dhcp6: false
      set-name: "ens192"
      addresses: [192.168.86.20/24]
      routes:
      - to: default
        via: 192.168.86.34
```

Then, run:
{% include codeblock.html code="netplan apply" %}

### Install docker

Refresh the Ubuntu server's package list with the latest information from the repositories.
{% include codeblock.html code="sudo apt update" %}
Install the docker package.
{% include codeblock.html code="sudo apt install docker.io -y" %}
Add the "ubuntu" user to the "docker" group
{% include codeblock.html code="sudo vi /etc/group" %} 
Then, make the group changes take effect on user "ubuntu" by logging out and then logging back in again.  Or, if you don't want to log out, then you can run the following command:
{% include codeblock.html code="exec newgrp docker" %} 

Run the following to confirm that docker is running fine.
{% include codeblock.html code="
sudo systemctl status docker
docker ps   # to confirm if ubuntu can run docker commands successfully
" %}



### Install the tanzu cli

Check out the steps from the official [doc](https://techdocs.broadcom.com/us/en/vmware-tanzu/standalone-components/tanzu-kubernetes-grid/2-5/tkg/install-cli.html#:~:text=the%20steps%20below.-,Install%20using%20a%20package%20manager,-To%20install%20the){:target="_blank"}.

Basically, these are the main commands to run to install the cli.
{% include codeblock.html code="
sudo apt update

sudo apt install -y ca-certificates curl gpg

sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://storage.googleapis.com/tanzu-cli-installer-packages/keys/TANZU-PACKAGING-GPG-RSA-KEY.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/tanzu-archive-keyring.gpg

echo \"deb [signed-by=/etc/apt/keyrings/tanzu-archive-keyring.gpg] https://storage.googleapis.com/tanzu-cli-installer-packages/apt tanzu-cli-jessie main\" | sudo tee /etc/apt/sources.list.d/tanzu.list

sudo apt update

sudo apt install tanzu-cli=1.3.0
" %}

Confirm that tanzu cli is running successfully, and what the version is by running:
{% include codeblock.html code="tanzu version" %}

e.g.,
```
$ tanzu version
version: v1.3.0
buildDate: 2024-05-09
sha: d59f47c8
arch: amd64
```

Install the plugins for TKGM v2.5.2:
{% include codeblock.html code="
tanzu plugin group get vmware-tkg/default:v2.5.2

tanzu plugin group search -n vmware-tkg/default --show-details

tanzu plugin install --group vmware-tkg/default:v2.5.2
" %}

Confirm the list of plugins that are installed:
{% include codeblock.html code="tanzu plugin list" %}

e.g.,
```
$ tanzu plugin list
  NAME                DESCRIPTION                                                        TARGET      INSTALLED  STATUS
  isolated-cluster    Prepopulating images/bundle for internet-restricted environments   global      v0.32.3    installed
  management-cluster  Kubernetes management cluster operations                           kubernetes  v0.32.3    installed
  package             tanzu package management                                           kubernetes  v0.32.1    installed
  pinniped-auth       Pinniped authentication operations (usually not directly invoked)  global      v0.32.3    installed
  secret              Tanzu secret management                                            kubernetes  v0.32.0    installed
  telemetry           configure cluster-wide settings for vmware tanzu telemetry         global      v1.1.1     installed
  telemetry           configure cluster-wide settings for vmware tanzu telemetry         kubernetes  v0.32.3    installed
```


### Install kubectl cli

Download the kubectl cli from Broadcom.  Go to TKGM product download page and find the *kubectl cli v1.28.11 for Linux* download item.  Download the file `kubectl-linux-v1.28.11+vmware.2.gz`.

Uncompress the file and install it:
{% include codeblock.html code="
gunzip kubectl-linux-v1.28.11+vmware.2.gz
sudo install kubectl-linux-v1.28.11+vmware.2 /usr/local/bin/kubectl
" %}
Run this command to verify the cli version:
{% include codeblock.html code="kubectl version" %}

e.g.,
```
$ kubectl version
Client Version: v1.28.11+vmware.2
...
```


### Install the govc cli

I will need to install the govc cli as well because I need to upload the TKGM OVA to vCenter, and then make it a template.  It's a little easier to do that task when you have the govc cli.

Check the latest version of the govc cli by going to `https://github.com/vmware/govmomi/releases/`.

Download the latest Linux x86 download file:

{% include codeblock.html code="
wget https://github.com/vmware/govmomi/releases/download/v0.50.0/govc_Linux_x86_64.tar.gz" %}

Uncompress the file and install the govc cli.
{% include codeblock.html code="
mkdir govc

cd govc

tar xvfz ../govc_Linux_x86_64.tar.gz

sudo install govc /usr/local/bin/govc

" %}

Run this command to verify the cli version:
{% include codeblock.html code="govc version" %}

e.g.,
```
$ govc version
govc 0.50.0

```

### Create the govcenv.sh file
Create a file named govcenv.sh with contents similar to the following:
{% include codeblock.html code="
export GOVC_URL=https://vc-1.deephackmode.io
export GOVC_USERNAME=administrator@vsphere.local
export GOVC_PASSWORD='password_here'
export GOVC_DATACENTER=Datacenter
export GOVC_CLUSTER=ClusterNSX
export GOVC_DATASTORE=datastore1
export GOVC_NETWORK=LS1.1
export GOVC_INSECURE=1
" %}

Source the file to set up the environment variables:
{% include codeblock.html code=". govcenv.sh" %}

Run this command to confirm that it's working:
{% include codeblock.html code="govc about" %}

e.g.,
```
$ . govcenv.sh
ubuntu@numbat:~$ govc about
FullName:     VMware vCenter Server 8.0.3 build-24022515
Name:         VMware vCenter Server
Vendor:       VMware, Inc.
Version:      8.0.3
Build:        24022515
OS type:      linux-x64
API type:     VirtualCenter
API version:  8.0.3.0
Product ID:   vpx
UUID:         9a3d245a-c9b4-4cbe-8e1c-92477159fcd1

```

Next up, we'll upload the TKGM OVA into vSphere and then create a management and a workload cluster.
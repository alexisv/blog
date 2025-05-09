---
layout: post
title:  "Installing Avi on vCenter"
date:   2025-05-08 23:29:00 -0400
categories: deephackmode.io update
---
Before I proceed with installing TKGM, I decided that I am going to add Avi Controller in the mix.  This post has the steps that I ran to install the Avi Controller.  

I followed the steps from the TKGM doc titled ["Install and Configure NSX Advanced Load Balancer"](https://techdocs.broadcom.com/us/en/vmware-tanzu/standalone-components/tanzu-kubernetes-grid/2-5/tkg/mgmt-reqs-network-nsx-alb-install.html){:target="_blank"}

### Download the Avi Controller OVA

I confirmed from the Interoperability Matrix that Avi Controller v22.1.3 is compatible with TKGM v2.5.2.  I downloaded that version of the OVA from the Broadcom Download portal.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-05-08-installing-avi/avi-lb-download.png" alt="Download Avi Controller OVA" title="Download Avi Controller OVA">
</div>
<figcaption>Download Avi Controller OVA</figcaption>
</figure> 



### Deploy the OVA template to vSphere
The TKGM doc redirects to the [Avi doc on how to deploy the OVA](https://techdocs.broadcom.com/us/en/vmware-security-load-balancing/avi-load-balancer/avi-load-balancer/22-1/vmware-avi-load-balancer-installation-guide/installing-nsx-alb-in-vmware-vsphere-environments/deploying-avi-load-balancer-controller-in-vmware-vcenter/deploying-avi-vantage-in-write-access-mode/deploying-avi-controller-ova.html){:target="_blank"}.  I followed what was in that doc, but roughly these were the steps:

1. In vCenter, right-click on the cluster object and then select "Deploy OVF Template".
1. In the "Select an OVF template" tab, select "Local file" and then upload the downloaded OVA file.  Then, click "Next".
1. In the "Select a name and folder" tab, leave the VM name as it is, which is `controller`.  Select the folder `pcf_vms` as the location.  Then, click "Next"
1. In the "Select a compute resource" tab, select the cluster and then click "Next".
1. In the "Review details" tab, just click "Next".
1. In the "Select storage" tab, click the Datastore, and then set the virtual disk format to "Thick Provision Lazy Zeroed".  Then, click "Next".
1. In the "Select networks" tab, set VM Network to "VM Network".  Then, click "Next".
1. In the "Customize template" tab, specify the management IP address and default gateway.  Then, click "Next".
1. In the "Ready to complete" tab, click "Finish".
1. Monitor the task in vCenter.  It should complete in just a few minutes.
1. Power on the VM.
1. Setup DNS entry for the Controller and open it in your browser to setup the admin credentials.

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-05-08-installing-avi/nsx-initial-login.png" alt="Avi Initial Auth setup" title="Avi Initial Auth setup">
</div>
<figcaption>Avi Initial Auth setup</figcaption>
</figure> 


### Performing the initial setup

I use the steps in the ["Performing the Avi Load Balancer Controller Initial setup"](https://techdocs.broadcom.com/us/en/vmware-security-load-balancing/avi-load-balancer/avi-load-balancer/22-1/vmware-avi-load-balancer-installation-guide/installing-nsx-alb-in-vmware-vsphere-environments/deploying-avi-load-balancer-controller-in-vmware-vcenter/deploying-avi-vantage-in-write-access-mode/performing-the-avi-controller-initial-setup.html){:target="_blank"} doc as a guide.

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-05-08-installing-avi/basic-settings-1.png" alt="Configure the Basic System Settings 1" title="Configure the Basic System Settings 1">
<img class="popup-img" src="/assets/images/2025-05-08-installing-avi/basic-settings-2.png" alt="Configure the Basic System Settings 2" title="Configure the Basic System Settings 2">
<img class="popup-img" src="/assets/images/2025-05-08-installing-avi/vmware-cloud-1.png" alt="Configure the VMware Cloud 1" title="Configure the VMware Cloud 1">
<img class="popup-img" src="/assets/images/2025-05-08-installing-avi/vmware-cloud-2.png" alt="Configure the VMware Cloud 2" title="Configure the VMware Cloud 2">
<img class="popup-img" src="/assets/images/2025-05-08-installing-avi/vmware-cloud-3.png" alt="Configure the VMware Cloud 3" title="Configure the VMware Cloud 3">
<img class="popup-img" src="/assets/images/2025-05-08-installing-avi/vmware-cloud-4.png" alt="Configure the VMware Cloud 4" title="Configure the VMware Cloud 4">
</div>
<figcaption>The screens from the Initial Setup</figcaption>
</figure> 

I used "vmware-cloud" as the name of the cloud, mainly because that was the example in the screenshot from the document although it was not mentioned at all.  Also, I didn't enable the DHCP, IPv6 Auto Configuration, and Content Library.

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-05-08-installing-avi/vmware-cloud-added.png" alt="VMware Cloud added" title="VMware Cloud added">
</div>
<figcaption>VMware Cloud added</figcaption>
</figure> 

### Avi Controller Setup: IPAM and DNS

As per the TKGM doc, there are additional settings that needed to be configured, such as the IPAM and DNS Profiles.

I followed the steps from the section ["Avi Controller Setup: IPAM and DNS"](https://techdocs.broadcom.com/us/en/vmware-tanzu/standalone-components/tanzu-kubernetes-grid/2-5/tkg/mgmt-reqs-network-nsx-alb-install.html#:~:text=1.5.2%2B-,Avi%20Controller%20Setup%3A%20IPAM%20and%20DNS,-There%20are%20additional){:target="_blank"}

The only thing confusing from that doc is that specifically instructed to select the "Default-Cloud" cloud, even though in the previous steps I created the "vmware-cloud" cloud as per the Avi document.  In those steps, I just select "vmware-cloud".

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-05-08-installing-avi/tkg-ipam-profile-1.png" alt="tkg-ipam-profile" title="tkg-ipam-profile">
<img class="popup-img" src="/assets/images/2025-05-08-installing-avi/tkg-dns-profile-1.png" alt="tkg-dns-profile" title="tkg-dns-profile">
<img class="popup-img" src="/assets/images/2025-05-08-installing-avi/ipam-dns-vmware-cloud.png" alt="Associate the profiles to the cloud" title="Associate the profiles to the cloud">
<img class="popup-img" src="/assets/images/2025-05-08-installing-avi/network-profiles.png" alt="The VM Network" title="The VM Network">
<img class="popup-img" src="/assets/images/2025-05-08-installing-avi/network-profiles-subnet.png" alt="Subnet Pool" title="Subnet Pool">
</div>
<figcaption>Adding the IPAM and DNS profiles, plus the subnet pool</figcaption>
</figure> 

### Create a dummy Virtual Service

The TKGM doc recommended to create a dummy virtual service to trigger the creation of a service engine before deploying a management cluster.  So, I followed this as well.  I just pointed the dummy service to an Ops Manager DNAT destination that was live.  I saw 2 Service Engines being spun up in vCenter.  I also see the number of Virtual Services turned to 1 after the Service Engines came up, and I gather that this is the indicator of success and that it's now ready for creating a TKGM Management Cluster.

<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2025-05-08-installing-avi/service-engines.png" alt="Service Engines" title="Service Engines">
<img class="popup-img" src="/assets/images/2025-05-08-installing-avi/service-engine-group.png" alt="Service Engines Group" title="Service Engines Group">
</div>
<figcaption>Service Engines and the number of Virtual Services</figcaption>
</figure> 
---
layout: post
title:  "Installing VCSA"
date:   2025-04-12 00:38:00 -0400
categories: deephackmode.io update
---
Now that we have an ESXi host, we can then deploy VCSA as a virtual machine.

VCSA stands for "VMware vCenter Server Appliance".  This is a virtual machine that is pre-configured and optimized for running the VMware vCenter Server and its services.

I have downloaded the VCSA installer ISO file from the Broadcom download portal.  The version I chose is `8.0.3`. The file name was:

```
VMware-VCSA-all-8.0.3-24022515.iso
```

Similar to the ESXi installation, this one is also simple and straightforward.

The following is the series of steps and the screens that I captured as the installation progressed to completion.  I used a Windows machine to mount the ISO file as a CD/DVD drive.

Using the File Explorer, find the ISO file and then right-click on it and click "Mount":
![Mount the ISO file](/assets/images/2025-04-12-installing-vcsa/dvd-1-mount.png "Mount the ISO file"){: width="1024" }{: .popup-img }

Click the DVD drive to view its contents.  Double-click the `vcsa-ui-installer` folder to open it and view its contents.
![DVD contents](/assets/images/2025-04-12-installing-vcsa/dvd-2-contents.png "DVD contents"){: width="1024" }{: .popup-img }

Open the `win32` folder, and then find the `installer` file and double-click it to launch it.
![Installer file](/assets/images/2025-04-12-installing-vcsa/dvd-3-installer.png "Installer file"){: width="1024" }{: .popup-img }

Welcome to the vCenter Server 8.0 Installer!  Click "Install" to continue.
![Welcome to the Installer](/assets/images/2025-04-12-installing-vcsa/install-01.png "filWelcome to the Installer"){: width="1024" }{: .popup-img }

See the Introduction and click "Next" to continue.
![Installer Intro](/assets/images/2025-04-12-installing-vcsa/install-02.png "Installer Intro"){: width="1024" }{: .popup-img }

Accept the EULA and click "Next".
![Accept EULA](/assets/images/2025-04-12-installing-vcsa/install-03.png "Accept EULA"){: width="1024" }{: .popup-img }

Enter the target ESXi host.
![Enter Target ESXi](/assets/images/2025-04-12-installing-vcsa/install-04.png "Enter Target ESXi"){: width="1024" }{: .popup-img }

Accept the certificate warning.
![Accept cert thumbprint](/assets/images/2025-04-12-installing-vcsa/install-05.png "Accept cert thumbprint"){: width="1024" }{: .popup-img }

Name the vCenter VM, and set the root password.
![vCenter Name](/assets/images/2025-04-12-installing-vcsa/install-06.png "vCenter Name"){: width="1024" }{: .popup-img }

Pick a "T-shirt size".  Tiny works for me as I foresee that I won't be deploying more than 100 VM's.
![VM size selection](/assets/images/2025-04-12-installing-vcsa/install-07.png "VM size selection"){: width="1024" }{: .popup-img }

Select the datastore to install vCenter.  Click "Enable Thin Disk Mode" for some efficiency on storage.
![Datastore selection](/assets/images/2025-04-12-installing-vcsa/install-08.png "Datastore selection"){: width="1024" }{: .popup-img }

Configure the Network Settings.  Configure the usual settings here.
![Network configuration](/assets/images/2025-04-12-installing-vcsa/install-09.png "Network configuration"){: width="1024" }{: .popup-img }

Review your settings, and then click "Finish" to continue.
![Review settings](/assets/images/2025-04-12-installing-vcsa/install-10.png "Review settings"){: width="1024" }{: .popup-img }

Watch paint dry.
![Installation started](/assets/images/2025-04-12-installing-vcsa/install-11.png "Installation started"){: width="1024" }{: .popup-img }

Just be patient.
![Installation progress](/assets/images/2025-04-12-installing-vcsa/install-12.png "Installation progress"){: width="1024" }{: .popup-img }

You have successfully deployed the vCenter Server!
![Deployment successful](/assets/images/2025-04-12-installing-vcsa/install-13.png "Deployment successful"){: width="1024" }{: .popup-img }

Continue to set up the vCenter Server.
![Stage 2 Intro](/assets/images/2025-04-12-installing-vcsa/stage2-01.png "Stage 2 Intro"){: width="1024" }{: .popup-img }

Set up NTP and enable ssh access.
![NTP and SSH](/assets/images/2025-04-12-installing-vcsa/stage2-02.png "NTP and SSH"){: width="1024" }{: .popup-img }

Set the credentials for the administrator account.
![Administrator credentials](/assets/images/2025-04-12-installing-vcsa/stage2-03.png "Administrator credentials"){: width="1024" }{: .popup-img }

Configure CEIP.
![CEIP](/assets/images/2025-04-12-installing-vcsa/stage2-04.png "CEIP"){: width="1024" }{: .popup-img }

Review your settings.  Click Finish to continue.
![Review Stage 2 settings](/assets/images/2025-04-12-installing-vcsa/stage2-05.png "Review Stage 2 settings"){: width="1024" }{: .popup-img }

Setup in progress.
![Setup in progress](/assets/images/2025-04-12-installing-vcsa/stage2-06.png "Setup in progress"){: width="1024" }{: .popup-img }

Stage 2 Set Up completed.
![Stage 2 complete](/assets/images/2025-04-12-installing-vcsa/stage2-07.png "Stage 2 complete"){: width="1024" }{: .popup-img }

Using your browser, go to your vCenter URL and log in with the administrator account.
![New vCenter](/assets/images/2025-04-12-installing-vcsa/vcenter-login-page.png "New vCenter"){: width="1024" }{: .popup-img }

Create a New Datacenter, by right-clicking on the vCenter object and selecting "New Datacenter".  I named it "Datacenter".
![New Datacenter](/assets/images/2025-04-12-installing-vcsa/vcenter-01.png "New Datacenter"){: width="1024" }{: .popup-img }

Create a Cluster.  Right-click on the "Datacenter" object and select "New Cluster".
![New Cluster](/assets/images/2025-04-12-installing-vcsa/vcenter-02.png "New Cluster"){: width="1024" }{: .popup-img }

Name your Cluster.  Enable DRS because it will be required for the TKGI foundation.  Don't enable HA because I only have 1 host.
![Configure cluster](/assets/images/2025-04-12-installing-vcsa/vcenter-03.png "Configure cluster"){: width="1024" }{: .popup-img }

Set the image to be used for the cluster.  This image should correspond with the ESXi version installed in the host.
![Set image](/assets/images/2025-04-12-installing-vcsa/vcenter-04.png "Set image"){: width="1024" }{: .popup-img }

Review and continue.
![Review cluster](/assets/images/2025-04-12-installing-vcsa/vcenter-05.png "Review cluster"){: width="1024" }{: .popup-img }

In the Cluster Quickstart page, the "Cluster basics" should now be completed and marked with a green checkmark.  Click "Add hosts".
![Cluster basics](/assets/images/2025-04-12-installing-vcsa/vcenter-06.png "Cluster basics"){: width="1024" }{: .popup-img }

Add the host to the cluster.
![Add host](/assets/images/2025-04-12-installing-vcsa/vcenter-07.png "Add host"){: width="1024" }{: .popup-img }

Review the security alert on the host certificate, and continue accordingly.
![SSL alert](/assets/images/2025-04-12-installing-vcsa/vcenter-08.png "SSL alert"){: width="1024" }{: .popup-img }

Review Host Summary and click "Next".
![Host Summary](/assets/images/2025-04-12-installing-vcsa/vcenter-09.png "Host Summary"){: width="1024" }{: .popup-img }

Select the host to import the image from.
![Import Image](/assets/images/2025-04-12-installing-vcsa/vcenter-10.png "Import Image"){: width="1024" }{: .popup-img }

Review and click "Finish" to continue.
![Review Add hosts](/assets/images/2025-04-12-installing-vcsa/vcenter-11.png "Review Add hosts"){: width="1024" }{: .popup-img }

In the Quickstart page, the "Add hosts" step should now be complete and marked with a green checkmark.  Also, the host should now appear under the cluster in the left pane.
![Host added](/assets/images/2025-04-12-installing-vcsa/vcenter-12.png "Host added"){: width="1024" }{: .popup-img }

In the Updates tab of the Cluster, click "Hosts->Image".  Click the "CHECK COMPLIANCE" button, and wait for the check to complete.  It should show that all the hosts in the cluster are compliant.
![Cluster Image compliance](/assets/images/2025-04-12-installing-vcsa/vcenter-13.png "Cluster Image compliance"){: width="1024" }{: .popup-img }

Assign a valid license for vCenter product.  Right-click on the vCenter object and click "Assign License".
![Assign License](/assets/images/2025-04-12-installing-vcsa/vcenter-assign-license.png "Assign License"){: width="256" }{: .popup-img }

Go to "New License" and add the License key.
![Add License](/assets/images/2025-04-12-installing-vcsa/vcenter-assign-new-license.png "Add License"){: width="768" }{: .popup-img }

Go to "Existing Licenses" and view the Licenses added in this vCenter instance.
![View Licenses](/assets/images/2025-04-12-installing-vcsa/vcenter-licenses.png "View Licenses"){: width="768" }{: .popup-img }

**Congratulations!  Your vCenter is now up and running!**

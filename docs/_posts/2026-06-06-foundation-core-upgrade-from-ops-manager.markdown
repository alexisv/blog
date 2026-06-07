---
layout: post
title:  "Foundation Core upgrade from Ops Manager"
date:   2026-006-06 23:57:00 -0400
categories: deephackmode.io update
---
I have uninstalled a few products in my homelab.  I basically left the NSX system, router VM and Ops Manager running.  My vCenter look like this:
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/vcenter.png" alt="vCenter UI" title="vCenter UI">
</div>
<figcaption>vCenter showing running VM's</figcaption>
</figure> 

The Ops Manager only has the Bosh Director tile.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/opsmanager.png" alt="Ops Manager page" title="Ops Manager page">
</div>
<figcaption>Ops Manager page showing only Bosh Director tile</figcaption>
</figure>

Clean slate!  My plan was to install the Elastic Application Runtime tile.  Before doing that, I upgraded the Ops Manager to the latest version 3.3.2.

To upgrade the Ops Manager, I exported the installation settings first and saved the file in my local.  This file contains all the information of the current Ops Manager installation, and will be imported into the new one later.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/export-installation-settings.png" alt="Export Installation Settings" title="Export Installation Settings">
</div>
<figcaption>Export Installation Settings</figcaption>
</figure>

I downloaded the newest Ops Manager OVA (v3.3.2) from the Broadcom Download site.  

In vCenter, I powered off the current Ops Manager VM.  I then right-clicked on my only resource pool item (RP01), and then selected "Deploy OVF Template".  I upload the OVA that I just downloaded:
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/deploy-ovf.png" alt="Deploy OVF Template" title="Deploy OVF Template">
</div>
<figcaption>Deploy OVF Template</figcaption>
</figure>

I entered name of the new VM as "ops-manager-3.3.2" accordingly, and selected the "pcf_vms" folder as location.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/select-name-and-folder.png" alt="Select a name and folder" title="Select a name and folder">
</div>
<figcaption>Select a name and folder</figcaption>
</figure>

I selected the only resource pool "RP01" as destination compute resource.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/select-compute-resource.png" alt="Select a compute resource" title="Select a compute resource">
</div>
<figcaption>Select a compute resource</figcaption>
</figure>

In Review Details, I confirmed that everything looks good so far and clicked Next.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/review-details.png" alt="Review details" title="Review details">
</div>
<figcaption>Review details</figcaption>
</figure>

Selected the only datastore I have as the storage.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/select-storage.png" alt="Select storage" title="Select storage">
</div>
<figcaption>Select storage</figcaption>
</figure>

Selected "LS1.1" as the destination network.  This is the same network set in the current Ops Manager VM.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/select-networks.png" alt="Select networks" title="Select networks">
</div>
<figcaption>Select networks</figcaption>
</figure>

In Customize template page, I entered the same settings as the current Ops Manager VM.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/customize-template-1.png" alt="Customize template part 1" title="Customize template part 1">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/customize-template-2.png" alt="Customize template part 2" title="Customize template part 2">
</div>
<figcaption>Customize template</figcaption>
</figure> 

In Ready to complete page, I verified all the details and then clicked "Finish".
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/ready-to-complete-1.png" alt="Ready to complete part 1" title="Ready to complete part 1">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/ready-to-complete.png" alt="Ready to complete part 2" title="Ready to complete part 2">
</div>
<figcaption>Ready to complete</figcaption>
</figure> 

The OVA started deploying.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/vcenter-deploying-ova.png" alt="OVA deployment status" title="OVA deployment status">
</div>
<figcaption>OVA deployment status</figcaption>
</figure>

After around 8 minutes, the deployment completed!
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/vcenter-deploy-completed.png" alt="OVA deployment completed" title="OVA deployment completed">
</div>
<figcaption>OVA deployment status shows that it completed</figcaption>
</figure>

Powered on the new Ops Manager VM, and waited for it to get an IP address assigned.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/opsmanager-powered-on.png" alt="Ops Manager powered on" title="Ops Manager powered on">
</div>
<figcaption>Ops Manager powered on and IP address assigned</figcaption>
</figure>

Navigated to the Ops Manager URL (https://opsmgr.deephackmode.io), and was greeted with a Welcome page.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/opsmanager-first-screen.png" alt="Ops Manager Welcome page" title="Ops Manager Welcome page">
</div>
<figcaption>Ops Manager Welcome page</figcaption>
</figure>

Clicked "Import Existing Installation", and uploaded the "installation.zip" file I saved earlier.  Also, obviously I entered the Decryption Passphrase, which is a must.  
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/upload-installation-zip.png" alt="Import the installation.zip file" title="Import the installation.zip file">
</div>
<figcaption>Import the installation.zip file</figcaption>
</figure>

Clicked the "Import" button, and it started processing the file.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/opsmanager-wait.png" alt="Waiting for Ops Manager to be ready" title="Waiting for Ops Manager to be ready">
</div>
<figcaption>Waiting for Ops Manager to be ready</figcaption>
</figure>

After a few seconds, the usual Login Page appeared.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/opsmanager-login.png" alt="Ops Manager Login Page" title="Ops Manager Login Page">
</div>
<figcaption>Ops Manager Login Page</figcaption>
</figure>

Logged in the with the 'admin' account and password, and then I was greeted with the new UI.  Also, it is now named "Foundation Core"!
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/opsmanager-new-foundation-core.png" alt="Welcome to Foundation Core" title="Welcome to Foundation Core">
</div>
<figcaption>Welcome to Foundation Core!</figcaption>
</figure>

Navigated to "Manage" tab and then to the "Capabilities" page, which shows the current tiles that are installed.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/foundation-core-capabilities.png" alt="Capabilities page" title="Capabilities page">
</div>
<figcaption>Capabilities page showing the installed tiles</figcaption>
</figure>

Clicked on "Changes" tab, and clicked on "Apply Pending Changes" to deploy the new Bosh Director version.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/foundation-core-review-pending-changes.png" alt="Apply Pending Changes" title="Apply Pending Changes">
</div>
<figcaption>Apply Pending Changes</figcaption>
</figure> 

Apply Changes running.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/foundation-core-applying-changes.png" alt="Apply Changes running" title="Apply Changes running">
</div>
<figcaption>Apply Changes running</figcaption>
</figure> 

It completed in around 8 minutes!
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-06-foundation-core-upgrade-from-ops-manager/foundation-core-apply-changes-completed.png" alt="Apply Changes completed" title="Apply Changes completed">
</div>
<figcaption>Apply Changes completed!</figcaption>
</figure> 


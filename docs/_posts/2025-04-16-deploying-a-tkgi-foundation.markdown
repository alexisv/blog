---
layout: post
title:  "Deploying a TKGI Foundation"
date:   2025-04-16 19:48:00 -0400
categories: deephackmode.io update
---
In the last few posts, I have paved my infrastructure, which comprises a vSphere environment with NSX.  The whole infrastructure is running on a single Dell PowerEdge R740 server.

In this post, I'm going to deploy a TKGI Foundation on top of the infrastructure.

Here are the main steps in this deployment:

1. Create the necessary objects in vSphere
1. Create the necessary objects in NSX
1. Deploy Tanzu Operations Manager v3.0.40
1. Deploy Bosh Director
1. Deploy Tanzu Kubernetes Grid Integrated Edition v1.21.0


### Create the necessary objects in vSphere

A few folders and a Resource Pool need to be created in vSphere before deploying the TKGI foundation.

1. In vCenter, go to the Hosts tab and the right-click on Cluster and click "New Resource Pool".  Add a new resource pool (e.g., `RP01`).  When adding the resource pool, only the name is required to be entered, the rest of the settings can be left as they are.  Save the new resource pool by clicking "OK".
    ![Resource Pool](/assets/images/2025-04-16-deploying-a-tkgi-foundation/resource_pool.png "Resource Pool"){: .popup-img }{: width="256" }

1. In the VMs and Templates tab, right-click on Datacenter and click "New Folder"->"New VM and Template Folder".  Enter `pcf_templates` as name and then click "OK".
1. Repeat the previous step to add `pcf_vms` folder.

    ![VM and Template Folders](/assets/images/2025-04-16-deploying-a-tkgi-foundation/pcf_folders.png "VM and Template Folders"){: .popup-img }{: width="192" }

1. In the Storage tab, click the particular datastore that you plan to use.  Then, click "Files" tab on the right pane.  Click "New Folder", and enter `pcf_disk` as the name, and then click "OK".
    ![Disk Folder](/assets/images/2025-04-16-deploying-a-tkgi-foundation/pcf_disk.png "Disk Folder"){: .popup-img }{: width="512" }


### Create the necessary objects in NSX

1. In NSX Manager, go to Networking->NAT and add the following NAT rules under the `T0-gateway-1` T0 gateway.

    | **Name** | **Action** | **Match** | **Translated IP** |
    | --- | --- | --- | --- |
    | opsmanager | DNAT | Destination IP: 192.168.16.10 | 10.1.1.10 |
    | snat1-1 | SNAT | Source IP: 10.1.1.0/24 | 192.168.16.1 |

    ![NAT Rules](/assets/images/2025-04-16-deploying-a-tkgi-foundation/nat-rules.png "NAT Rules"){: .popup-img }{: width="1024" }

1. Go to Networking->IP Address Pools->IP Address Pools and add the following pool.
- `floating-ip-pool` - (Range: `192.168.16.101-192.168.16.254`, CIDR: `192.168.16.0/24`) 

1. Go to Networking->IP Address Pools->IP Address Blocks and add the following block.
- `pods-ip-block` - (CIDR: `172.16.0.0/16`)
- `nodes-ip-block` - (CIDR: `10.16.0.0/16`)






### Deploy the Tanzu Operations Manager

1. Download the OVA from the Broadcom Support Portal
1. In vCenter, right-click on the Resource Pool "RP01", and the click "Deploy OVF Template".
1. In the "Select an OVF Template" screen, select "Local file" and then click on "Upload Files" to upload the OVA file `ops-manager-vsphere-3.0.40+LTS-T.ova` that was downloaded.  Click "Next".
1. In the "Select a name and folder" screen, enter a name of the VM or accept the default.  Select the "pcf_vms" folder as the location for the VM.  Click "Next".
1. Review the details and then click "Next".
1. Select the storage that you want to use, and then click "Next".
1. Select the network "LS1.1" and then click "Next".
1. In the Customize template screen, provide the following info, and then click "Next".

    | **Setting** | **Value** |
    | --- | --- |
    | IP Address | 10.1.1.10 |
    | Netmask | 255.255.255.0 |
    | Default Gateway | 10.1.1.1 |
    | DNS | 192.168.86.34 |
    | NTP Servers | 192.168.86.34 |
    | Public SSH Key | your ssh key here |
    | Custom Hostname | opsmgr |

1. Review the details and then click "Finish".
1. Check the deployment task in vCenter.  Wait for it to complete.
1. Set up DNS to add the A entry for Ops Manager's DNAT Destination IP (`192.168.16.10`) and FQDN `opsmgr.deephackmode.io`.
1. In a browser, go to `https://opsmgr.deephackmode.io`.
1. Set up "Internal Authentication".  Setup username and password for the `admin` account.  Provide a passphrase as well.  Click "OK" to complete the authentication setup.
1. At the Ops Manager Login, enter the `admin` user and password.

### Deploy the Bosh Director

1. In the Ops Manager UI, click the Director tile to get to the settings page.
1. In vCenter Config tab, provide the following info, and then click "Save".

    | **Setting** | **Value** |
    | --- | --- |
    | vCenter Name | vc-1 |
    | vCenter Host | vc-1.deephackmode.io |
    | vCenter Username | $username |
    | vCenter Password | $password |
    | Datacenter Name | Datacenter |
    | Virtual Disk Type | Thin |
    | Ephemeral Datastore Names | datastore1 |
    | Persistent Datastore Names | datastore1 |

    Click the "NSX-T Networking" radio box, and provide the following info:

    | NSX Address | nsx-mgr-1.deephackmode.io |
    | NSX-T Authentication | Local User Authentication |
    | NSX Username | $username |
    | NSX Password | $password |
    | Use NSX-T Policy API | :checked: |
    | NSX CA Cert | $NSXCA |
1. In Director Config tab, enter `192.168.86.34` as the NTP server.  The rest of the settings in this tab can be left with the default values.  Click "Save".
1. In Create Availability Zones tab, add 1 AZ with the following info, and then click "Save".

    | **Setting** | **Value** |
    | --- | --- |
    | Name | az1 |
    | IaaS Configuration | vc-1 |
    | Cluster | Cluster |
    | Resource Pool | RP01 |

1. In Create Networks tab, add 1 Network with the following info, and then click "Save".

    | **Setting** | **Value** |
    | --- | --- |
    | Name | deployment-network |
    | vSphere Network Name | LS1.1 |
    | CIDR | 10.1.1.0/24 |
    | Reserved IP Ranges | 10.1.1.0-10.1.1.10 |
    | DNS | 192.168.86.34 |
    | Gateway | 10.1.1.1 |
    | Availability Zones | az1 |

1. In Assign AZs and Networks, set the Singleton AZ to `az1` and the Network to `deployment-network`.  Click "Save".
1. In Security tab, enable the option "Include Tanzu Ops Manager Root CA in Trusted Certs".  Click "Save".
1. The rest of the tabs can be left with the default settings.
1. Click "Installation Dashboard" from the top menu, and then click "Review Pending Changes", and then click "Apply Changes".  Wait for the "Apply Changes" to complete.




### Deploy the Tanzu Kubernetes Grid Integrated Edition

1. Download the TKGI tile from the Broadcom Support Portal.
1. In Ops Manager, click "Import a Product" and then find the TKGI tile file `pivotal-container-service-1.21.0-build.32.pivotal` and upload it.
1. Once uploaded, the product would be listed in the left pane.  Click the `+` sign beside the product to stage it.
1. Once staged, the tile would appear on the main pane.  Click the product to begin configuring it.
1. In Assign AZs and Networks, Select `az1` on both "Place singleton jobs in AZ" and "Balance other jobs in AZs" settings.  Select `deployment-network` on both "Network" and "Service Network" settings.  Click "Save".
1. In TKGI API tab, click "Change" under the "Certificate to secure the TKGI API", and then Generate the cert and key for `*.deephackmode.io` name.
1. Enter `tkgi.deephackmode.io` as the API Hostname.  Click "Save".
1. In Plan 1 tab, check the `az1` checkbox under Master/ETCD Availability Zones and also under Worker Availability Zones.  The rest of the settings can be left as default.  Click "Save".
1. In Kubernetes Cloud Provider tab, enter the vCenter information.  The Stored VM Folder can be set to `pcf_vms`.  Click "Save".
1. In Networking tab, select "NSX-T" as the Container Networking Interface.
1. Follow the instructions on [how to generate and register a NSX Principal Identity](https://techdocs.broadcom.com/us/en/vmware-tanzu/standalone-components/tanzu-kubernetes-grid-integrated-edition/1-21/tkgi/nsxt-generate-pi-cert.html#certificate-super-user-script){:target="_blank"}.  Then, enter the generated Principal Identity cert and key into the NSX Manager Super User Principal Identity Certificate fields.
1. Enter the NSX Manager CA cert.  Get this from NSX Manager UI or retrieve using `openssl` cli (if it's using a self-signed server cert).
1. Enable NAT mode.
1. Enable Policy API mode.
1. Enter `pods-ip-block` as the Pods IP Block ID.
1. Enter `nodes-ip-block` as the Nodes IP Block ID.
1. Enter `T0-gateway-1` as the T0 Router ID.
1. Enter `floating-ip-pool` as the Floating IP Pool ID.
1. Enter `192.168.86.34` as the Nodes DNS.
1. Enter `Cluster` as the vSphere Cluster Names.
1. Click Save.
1. In CEIP tab, click "No" under "Join the... Program".  Select "Demo or POC" as your use-case. Click "Save".
1. In Storage tab, select "Yes" under vSphere CSI Driver Integration. Click "Save".
1. The rest of the tabs can be left with the default settings.
1. Click "Installation Dashboard" from the top menu, and then click "Review Pending Changes", and then click "Apply Changes".  Wait for the "Apply Changes" to complete.
    ![TKGI Foundation](/assets/images/2025-04-16-deploying-a-tkgi-foundation/tkgi-foundation.png "TKGI Foundation"){: .popup-img }{: width="1024" }

That right there is my brand new shiny TKGI Foundation!
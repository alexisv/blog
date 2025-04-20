---
layout: post
title:  "NSX Integration"
date:   2025-04-12 11:54:00 -0400
categories: deephackmode.io update
---
At this point, I now have a functioning vSphere environment.  Key components of vSphere include the ESXi hypervisor and vCenter Server.  Both of which are now up & running in my Homelab that has only one Dell PowerEdge R740 server, a tiny TP-Link 5-Port Gigabit switch, and a Raspberry PI.  The switch is connected to my main switch and to the NIC ports of the R740.  The Raspberry PI functions as a router, DNS, NTP and VPN server.

I am now going to deploy a NSX Manager and two Edge Node VM's in my Homelab.

I have downloaded the NSX Unified Appliance OVA.

```
nsx-unified-appliance-4.2.1.3.0.24533887.ova
```

I followed the official [NSX Quickstart guide](https://techdocs.broadcom.com/us/en/vmware-cis/nsx/vmware-nsx/4-2/quick-start-guide/installing-nsx-t.html){:target="_blank"} to install it.

The NSX Policy API is the default mode, and that is fine and I won't change it to Manager mode.  Later on, I will create the objects using Policy API and so I will use Policy API when I install the TKGI tile.

To prepare for the installation, I filled out this table with the planned network settings.

| **Setting** | **Notes** | **Your Value** |
| --- | --- | --- |
| VLAN 11 | For management traffic | 192.168.86.0/24 |
| VLAN 12 | For NSX overlay traffic | 192.168.20.0/24 |
| VLAN 50 | For traffic between the tier-0 gateway and physical router | 192.168.50.0/24 |
| PG-mgmt | VC dvportgroup backed by VLAN 11 (management VLAN) | "VM Network" |
| Management subnet | 192.168.10.0/24, default gateway: 192.168.10.1, subnet mask: 255.255.255.0 | 192.168.86.0/24, default gateway: 192.168.86.1, subnet mask: 255.255.255.0 |
| TEP (tunnel endpoint) subnet | 192.168.20.0/24, default gateway: 192.168.20.1, subnet mask: 255.255.255.0 | 192.168.20.0/24, default gateway: 192.168.20.1, subnet mask: 255.255.255.0 |
| VC IP address | 192.168.10.10 (VLAN 11) | 192.168.86.101 (VLAN 11) |
| ESXi-1 IP address | 192.168.10.11 (VLAN 11), 192.168.20.11 (VLAN 12) | 192.168.86.23 (VLAN 11), 192.168.20.11 (VLAN 12) |
| NSX-mgr-1 IP address | 192.168.10.14 (VLAN 11) | 192.168.86.251 (VLAN 11) |
| Edge-1 IP address | 192.168.10.17 (VLAN 11), 192.168.20.17 (VLAN 12) | 192.168.86.252 (VLAN 11), 192.168.20.17 (VLAN 12) |
| Edge-2 IP address | 192.168.10.18 (VLAN 11), 192.168.20.18 (VLAN 12) | 192.168.86.253 (VLAN 11), 192.168.20.18 (VLAN 12) |
| Physical router's downlink IP address | 192.168.50.1 (VLAN 50) | 192.168.50.1 (VLAN 50) |
| Tier-0 gateway's external interface IP address on Edge-1 | 192.168.50.11 (VLAN 50) | 192.168.50.11 (VLAN 50) |
| Tier-0 gateway's external interface IP address on Edge-2 | 192.168.50.12 (VLAN 50) | 192.168.50.12 (VLAN 50) |
| Tier-0 gateway's virtual IP (VIP) | 192.168.50.13 (VLAN 50) | 192.168.50.13 (VLAN 50) |
| Segment 1 subnet | 10.1.1.0/24 | 10.1.1.0/24 |
| Segment 2 subnet | 10.1.2.0/24 | 10.1.2.0/24 |
| Segment 3 subnet | 10.2.1.0/24 | 10.2.1.0/24 |
| Test-VM-1 IP address | 10.1.1.11 | 10.1.1.11 |
| Test-VM-2 IP address | 10.1.2.11 | 10.1.2.11 |
| Test-VM-3 IP address | 10.2.1.11 | 10.2.1.11 |

### Step 1: Deploy the NSX Manager {#step-1}

The NSX manager VM is deployed through VC's Deploy OVF Template wizard.

1. In VC, right click Cluster and select Deploy OVF Template.
1. Follow the prompts and provide the following information.

    | Location of the ova file | The location where you downloaded the ova file. |
    | Virtual machine name | NSX-mgr-1 |
    | Location for the VM | Cluster |
    | Compute resource | 192.168.86.23 (ESX-1's IP address) |
    | Deployment configuration | Medium |
    | Storage | datastore1 |
    | Virtual disk format | Thin Provision |
    | Network 1 | VM Network |
    | IP allocation | Static Manual |
    | IP protocol | IPv4 |
    | Hostname | nsx-mgr-1.deephackmode.io |
    | Rolename | NSX Manager |
    | Default IPv4 Gateway | 192.168.86.34 |
    | Management Network IPv4 Address | 192.168.86.251 |
    | Management Network Netmask | 255.255.255.0 |
    | DNS Server list | 192.168.86.34 |
    | NTP Server list | 192.168.86.34 |

1. Wait for the deployment to finish. The Recent Tasks panel at the bottom of the vSphere Client window will indicate when the task is complete.
1. Power on the NSX Manager VM.
1. Set up the DNS record for the NSX Hostname.
1. Log in to NSX-mgr-1 at `https://nsx-mgr-1.deephackmode.io`.  

    ![NSX Tour](/assets/images/2025-04-12-nsx-integration/welcome-to-nsx.png "NSX Tour"){: .popup-img }{: width="398" }

1. Go to System->Licenses and click Add to add your NSX license.
1. Go to System->Fabric->Compute Managers and click Add to add the VC as a compute manager. Follow the prompts and provide the necessary information about the vCenter.
1. Click Add at the warning Thumbprint is Missing.
1. Wait until Registration Status is Registered. You can click Refresh to refresh the status.

    ![Compute Manager](/assets/images/2025-04-12-nsx-integration/compute-manager.png "Compute Manager"){: .popup-img }{: width="1024" }


### Step 2: Configure a VDS {#step-2}

You can use an existing VDS or configure a new one. If you use an exising VDS, you must set its MTU to 1600 or higher.

#### Create a VDS in VC
1. In VC, under Networking, right click the datacenter and select Distributed Switch->New Distributed Switch.
1. Follow the prompts and provide the following information.

    | Name | VDS-NSX |
    | Location | Datacenter |
    | Spefify a distributed switch version | Select the version for your vSphere environment. |
    | Number of uplinks | 1 |
    | Network I/O Control | Enabled |

1. After VDS-NSX is created, right click it and select Add and Manage Hosts.
1. Select the ESXi host (192.168.86.23).
1. In the Manage physical adapters step, for each host, map vmnic1 to Uplink 1 (the default uplink name) on VDS-NSX.

#### Change the MTU value for VDS-NSX
1. In VC, under Networking, right click VDS-NSX and select Settings->Edit Settings.
1. In the MTU (Bytes) field, enter 1600.

#### Create a port group in VDS-NSX
1. In VC, under Networking, right click VDS-NSX and select New Distributed Port Group.
1. Provide the following information.

    Name | PG-all-VLAN |
    VLAN type | VLAN trunking |
    VLAN trunk range | 0-4094 |


### Step 3: Create an Uplink Profile and Configure Host Transport Nodes {#step-3}

#### Create an uplink profile
1. In NSX Manager, go to System->Fabric->Profiles->Uplink Profiles.
1. Click Add.
1. In the Name field, enter Uplink-profile-1.
1. Under Teamings, select Default Teaming and enter uplink1 for Active Uplinks.
1. In the Transport VLAN field, enter 12.

#### Create an IP Pool for the Host TEP network
1. In NSX Manager, go to Networking->IP Address Pools.
1. Click Add IP Address Pool.
1. Add "host-tep-pool" as the Name.
1. Click "Set" under the Subnets column.
1. Click Add Subnet.
1. Click "IP Ranges".
1. Add the IP Range "192.168.20.11-192.168.20.11".
1. Add the CIDR "192.168.20.0/24".
1. Click Apply to save the Subnet configuration.
1. Click Save to save the IP Pool.

#### Configure ESXi host transport nodes
1. In NSX Manager, go to System->Fabric->Hosts.
1. In the Transport Node Profile tab, click "Add Transport Node Profile"
1. Enter the Name as "tn-profile".
1. Click Host Switch->Set.  Then, click "Add Host Switch".
1. Select "vc-1" as vCenter.
1. Select "nsx-overlay-transportzone" and "nsx-vlan-transportzone" as the "Transport Zones".
1. Select "Uplink-profile-1" as the Uplink Profile.
1. Select "VDS-NSX" as the VDS.
1. Select "Use IP Pool" as the IPv4 Assignment.
1. Select "host-tep-pool" as the IP pool.
1. In the "Teaming Policy Uplink Mapping", select "Uplink 1" as the VDS Uplink to map to uplink1.
1. Click "Add", to save the Host Switch configuration.
1. Click "Apply", to add the Host Switch to the profile.
1. Click "Save", to add the Transport Node Profile.
1. In the Clusters tab, click the check box on the left side of the cluster.
1. Click "Configure NSX" from the menu above the table.
1. Select "tn-profile" as the Transport Node Profile, then click Save.
1. Wait until the NSX Configuration column displays Success. You can click the Refresh button to refresh the window.
    ![Host Transport Nodes](/assets/images/2025-04-12-nsx-integration/host-transport-nodes.png "Host Transport Nodes"){: .popup-img }{: width="1024" }


### Step 4: Deploy NSX Edge Nodes and Create an Edge Cluster {#step-4}

#### Create two segments for each Edge TEP network
1. In NSX Manager, go to Networking->Segments.
1. In NSX tab, click Add Segment.
1. Name the segment "Edge-uplink-1".
1. Leave the Connected Gateway as None.
1. Set the Transport Zone to nsx-vlan-transportzone.
1. Set VLAN to "0-4094"
1. Click Save to save the segment.
1. Repeat the steps to add another segment with name of "Edge-uplink-2".
    ![Segments](/assets/images/2025-04-12-nsx-integration/segments.png "Segments"){: .popup-img }{: width="1024" }

#### Create an IP Pool for the Edge TEP network
1. In NSX Manager, go to Networking->IP Address Pools.
1. Click Add IP Address Pool.
1. Add "edge-tep-pool" as the Name.
1. Click "Set" under the Subnets column.
1. Click Add Subnet.
1. Click "IP Ranges".
1. Add the IP Range "192.168.20.17-192.168.20.18".
1. Add the CIDR "192.168.20.0/24".
1. Add Gateway "192.168.20.1".
1. Click Apply to save the Subnet configuration.
1. Click Save to save the IP Pool.

#### Create two uplink profiles for each Edge node
1. In NSX Manager, go to System->Fabric->Profiles->Uplink Profiles.
1. Click Add.
1. In the Name field, enter Edge-uplink-1.
1. Under Teamings, select Default Teaming and enter uplink1 for Active Uplinks.
1. In the Transport VLAN field, enter 12.
    ![Edge Uplink Profiles](/assets/images/2025-04-12-nsx-integration/edge-uplink-profiles.png "Edge Uplink Profiles"){: .popup-img }{: width="1024" }

#### Deploy NSX Edge Nodes
1. In NSX Manager, go to System->Fabric->Nodes->Edge Transport Nodes.
1. Click Add Edge Node.
1. Provide the following information.

    | Name | Edge-1
    | Host name | edge-1.deephackmode.io
    | Form Factor | Select the appropriate edge node size.
    | CLI User Name | admin
    | CLI Password | 	
    | Allow SSH Login | Select an option based on your datacenter policy.
    | System Root Password |	
    | Allow Root SSH Login | Select an option based on your datacenter policy.
    | Audit User Name | audit
    | Audit Password | 	
    | Compute Manager | VC-1
    | Cluster | Cluster
    | Host | 192.168.86.23
    | Datastore | datastore1
    | IP Assignment | Static
    | Management IP | 192.168.86.252
    | Default Gateway | 192.168.86.34
    | Management Interface | "VM Network"
    | DNS Servers | 192.168.86.34
    | NTP Servers | 192.168.86.34
    | Edge Switch name | nvds1
    | Transport Zone | nsx-overlay-transportzone, nsx-vlan-transport-zone
    | Uplink Profile | Edge-uplink-1
    | IP Assignment | Use IP Pool
    | IP Pool |	edge-tep-pool
    | DPDK Fastpath Interfaces | Click Select Interface and select the VLAN Segment "Edge-uplink-1"

    ![Edge-1](/assets/images/2025-04-12-nsx-integration/edge-1.png "Edge-1"){: .popup-img }{: width="512" }

1. Wait until the Configuration State column displays Success. You can click the Refresh button to refresh the window.
1. Repeat steps 2-4 to deploy Edge-2, using the following information.  

   | Name | Edge-2
   | Host name | edge-2.deephackmode.io
   | Form Factor | Select the appropriate edge node size.
   | CLI User Name | admin
   | CLI Password | 	
   | Allow SSH Login | Select an option based on your datacenter policy.
   | System Root Password |	
   | Allow Root SSH Login | Select an option based on your datacenter policy.
   | Audit User Name | audit
   | Audit Password | 	
   | Compute Manager | VC-1
   | Cluster | Cluster
   | Host | 192.168.86.23
   | Datastore | datastore1
   | IP Assignment | Static
   | Management IP | 192.168.86.253
   | Default Gateway | 192.168.86.34
   | Management Interface | "VM Network"
   | DNS Servers | 192.168.86.34
   | NTP Servers | 192.168.86.34
   | Edge Switch name | nvds1
   | Transport Zone | nsx-overlay-transportzone, nsx-vlan-transport-zone
   | Uplink Profile | Edge-uplink-2
   | IP Assignment | Use IP Pool
   | IP Pool |	edge-tep-pool
   | DPDK Fastpath Interfaces | Click Select Interface and select the VLAN Segment "Edge-uplink-2"

    ![Edge-2](/assets/images/2025-04-12-nsx-integration/edge-2.png "Edge-2"){: .popup-img }{: width="512" }

1. Wait until the Configuration State column displays Success

#### Create an Edge Cluster
1. In NSX Manager, go to System->Fabric->Nodes->Edge Clusters.
1. Click Add Edge Cluster.
1. In the Name field, enter "Edge-cluster-1".
1. Move Edge-1 and Edge-2 from the Available window to the Selected window.
1. Click Add.


### Step 5: Configure Gateways and Segments {#step-5}

#### Create a VLAN Segment to Connect to the Physical Router
1. In NSX Manager, go to Networking->Segments.
1. Click Add Segment.
1. Provide the following information.

    | Segment Name | External-segment-1 |
    | Connected Gateway | None | 
    | Transport Zone | nsx-vlan-transportzone |
    | VLAN | 50 |

#### Create a Tier-0 Gateway
1. In NSX Manager, go to Networking->Tier-0 Gateways.
1. Click Add Tier-0 Gateway.
1. Enter a name for the gateway, for example, T0-gateway-1.
1. Select the HA (high availability) mode Active Standby.
1. Select the Edge cluster Edge-cluster-1.
1. Click Save and continue configuring this gateway.
1. Click Interfaces and click Set.
1. Click Add Interface.
1. Enter a name, for example, IP1-EdgeNode1.
1. Enter the IP address 192.168.50.11/24.
1. In the Connected To (Segment) field, select External-segment-1.
1. In the Edge Node field, select Edge-1.
1. Save the changes.
1. Repeat steps 8-13 to configure a second interface called IP2-EdgeNode2. The IP address should be 192.168.50.12/24. The Edge Node should be Edge-2.
1. In the HA VIP Configuration field, click Set to create a virtual IP for the tier-0 gateway.
1. Enter the IP address 192.168.50.13/24.
1. Select the interfaces IP1-EdgeNode1 and IP2-EdgeNode2.
1. Save the changes.
1. Configure Routing on the Physical Router and Tier-0 Gateway
1. On the physical router, configure a static route to the subnets 10.1.1.0/24, 10.1.2.0/24, and 10.2.1.0/24 via 192.168.50.13, which is the virtual IP address of the tier-0 gateway's external interface.
1. In NSX Manager, go to Networking->Tier-0 Gateways.
1. Edit T0-gateway-1.
1. Under Routing->Static Routes, click Set and click Add Static Route.
1. In the Name field, enter default.
1. In the Network field, enter 0.0.0.0/0.
1. Click Set Next Hops.
1. In the IP Address field, enter 192.168.50.1.
1. Click Add.
1. Save the changes.

#### Create Two Tier-1 Gateways
1. In NSX Manager, go to NetworkingTier-1 Gateways.
1. Click Add Tier-1 Gateway.
1. Provide the following information.

    | Tier-1 Gateway Name | T1-gateway-1 |
    | Edge Cluster | Edge-cluster-1 |
    | Linked Tier-0 Gateway | T0-gateway-1 |
1. Under Route Advertisement, enable All Connected Segments & Service Ports.
1. Save the changes.
1. Repeat steps 2-5 and create T1-gateway-2. Specify the same edge cluster.

#### Create Three Overlay Segments for VMs
1. In NSX Manager, go to Networking->Segments.
1. Click Add Segment.
1. Provide the following information.

    | Segment Name | LS1.1 |
    | Connected Gateway | T1-gateway-1 |
    | Transport Zone | nsx-overlay-transportzone |
    | Subnets | 10.1.1.1/24 |

    Note: For an overlay segment that is attached to a tier-1 gateway, in the Subnets field, specify an IP address for the tier-1 gateway. This address will be the default gateway for VMs attached to this segment.
1. Repeat steps 2-3 and create LS1.2 (Subnets: 10.1.2.1/24, Connected Gateway: T1-gateway-1) and LS2.1 (Subnets: 10.2.1.1/24, Connected Gateway: T1-gateway-2).
1. Verify that LS1.1, LS1.2, and LS2.1 are created under the appropriate VDS in VC.

At this point, check the Edge nodes at Fabric->Nodes->Edge Transport Nodes.  The nodes should have at least 2 tunnels showing up (green).
![Edge Transport Nodes](/assets/images/2025-04-12-nsx-integration/edge-transport-nodes.png "Edge Transport Nodes"){: .popup-img }{: width="1024" }


### Step 6: Test East-West and North-South Connectivity {#step-6}
1. Deploy a test VM in each of the 3 segments.
1. Assign each one an IP address
1. From each VM, ping the other Test-VM's and make sure that they are successful, and if so then that means that East-West connectivity is working.
1. From each VM, ping the Tier-0 gateway IP address 192.168.50.13 and external router (192.168.50.1).  If that was successful, then that meanst that North-South connectivity is working.
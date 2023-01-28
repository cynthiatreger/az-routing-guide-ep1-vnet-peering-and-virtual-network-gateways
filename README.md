# Episode #1: Discovery/Reminder of Azure VNET Connectivity & Impact of Virtual Network Gateways

[1.1. Discovery/Reminder of Azure VNET connectivity principles](https://github.com/cynthiatreger/az-routing-guide-part1-vnet-peering-and-virtual-network-gateways/edit/main/README.md#11-discoveryreminder-of-azure-vnet-connectivity-principles)

[1.1.1. VNET peering & Effective routes](https://github.com/cynthiatreger/az-routing-guide-ep1-vnet-peering-and-virtual-network-gateways/edit/main/README.md#111-vnet-peering--effective-routes)
  
[1.1.2. Standard H&S topology and non-transitivity of VNET peerings](https://github.com/cynthiatreger/az-routing-guide-ep1-vnet-peering-and-virtual-network-gateways/edit/main/README.md#112standard-hs-topology-and-non-transitivity-of-vnet-peerings)

[1.2. Connectivity Impact of adding a Virtual Network Gateway (ER or VPN)](https://github.com/cynthiatreger/az-routing-guide-part1-vnet-peering-and-virtual-network-gateways/edit/main/README.md#12-connectivity-impact-of-adding-a-virtual-network-gateway)

## 1.1. Discovery/Reminder of Azure VNET Connectivity Principles

### 1.1.1. VNET Peering & *Effective Routes*

To start, we will consider [2 peered VNETs](https://github.com/cynthiatreger/az-routing-guide-ep1-vnet-peering-and-virtual-network-gateways/edit/main/README.md#111-vnet-peering--effective-routes) and a VM in Spoke1 VNET. A single IP range* is allocated to each VNET. In the example below, each IP range is broken into 2 subnets.

(* an Azure Virtual Network (VNET) is made of one or multiple IP ranges and each range is split into subnets.)

<img width="610" alt="image" src="https://user-images.githubusercontent.com/110976272/215267844-9dd42db4-253d-4de9-a9d9-b348e6e37b75.png">

From an Azure networking perspective, according to Spoke1VM's *Effective routes*, Spoke1VM will know about the overall range of its own VNET (but won’t have the granularity of the more specific subnets) and the range of any other peered VNET.

By default all resources in a VNET can communicate outbound to the internet, hence the default route.

### 1.1.2. Standard Hub & Spoke Topology and Non-Transitivity of VNET Peerings

**Peering connections are non-transitive:** in a standard Hub & Spoke topology like depicted below, Spoke1VM will know about the Spoke1 VNET range AND the central VNET (Hub VNET) range, but because there is no direct peering between Spoke1 VNET and Spoke2 VNET, they won’t know about each other. 

Using this peering logic, since the Hub VNET is peered with both Spoke1 and Spoke2 VNETs, the HubTestVM will know about its own Hub VNET range and both Spoke1 & Spoke2 ranges.

<img width="1024" alt="image" src="https://user-images.githubusercontent.com/110976272/215268136-947d048a-c805-4bf5-b637-1ed2db7e563e.png">

## 1.2. Connectivity Impact of adding a Virtual Network Gateway (ER or VPN)

### 1.2.1.	Azure Virtual Network GW (VPN or ER) for On-Prem Connectivity
Whether it is VPN or ER, if a Virtual Network Gateway (VNG) is added in a VNET, the On-Prem routing prefixes received on the virtual Network GW will by default automatically be propagated and programmed in the Effective routes of any VM in this VNET and, **depending on per-peering settings**, in peered VNETs too. 

The VNET hosting the virtual Network GW is usually a central hub VNET, its range is advertised in return to the On-Prem.

Let's look at the **default route propagation** when an Virtual Network GW is deployed (Expressroute GW here, but the results would be similar with a VPN GW). By default, adding a Virtual Network GW does NOT enable the transit on EXISTING VNET peerings:

*The On-Prem is emulated by a VNET connected to the Hub VNET Expressroute circuit.* 

<img width="1024" alt="image" src="https://user-images.githubusercontent.com/110976272/215268623-5e4ca81d-fc4f-49f9-8607-f7c9a986f57d.png">

### 1.2.2.	Propagation of Spokes & On-Prem prefixes

#### “GW transit” scope = the entire VNET & the VNET range is not advertised On-Prem

Let's enable **GW Transit** on the peering between Spoke1 VNET and the Hub VNET (hosting the Virtual Network GW). Both sides of the peering must be updated:

<img width="800" alt="image" src="https://user-images.githubusercontent.com/110976272/215270865-bd19eb5f-1c29-4005-873a-e64b5260e56f.png">

:arrow_right: The On-Prem prefixes will start being “treated” like IP ranges of the Hub VNET: following the VNET peering principle they will get propagated to this directly peered Spoke VNET and be programmed in its VMs, with “Virtual Network Gateway” displayed as Next-Hop type.

:arrow_right: And in return, the Spoke1 VNET range will also be advertised On-Prem.

<img width="1024" alt="image" src="https://user-images.githubusercontent.com/110976272/215272200-057f63f8-b395-492c-863c-fec654c8ea0e.png">



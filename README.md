### [<< BACK TO THE MAIN MENU](https://github.com/cynthiatreger/az-routing-guide-intro)
##
# Episode #1: VNET connectivity, impact of Virtual Network Gateways, On-Prem route propagation options

*Introduction note: This guide aims at providing a better understanding of the Azure routing mechanisms and how they translate from On-Prem networking. The focus will be on private routing in Hub & Spoke topologies. For clarity, network security and resiliency best practices as well as internet breakout considerations have been left out of this guide.*
##
[1.1. Discovery/Reminder of Azure VNET connectivity principles](https://github.com/cynthiatreger/az-routing-guide-ep1-vnet-peering-and-virtual-network-gateways#11-discoveryreminder-of-azure-vnet-connectivity-principles)

&emsp;[1.1.1. VNET peering & *Effective routes*](https://github.com/cynthiatreger/az-routing-guide-ep1-vnet-peering-and-virtual-network-gateways#111-vnet-peering--effective-routes)
  
&emsp;[1.1.2. Standard Hub & Spoke topology and non-transitivity of VNET peerings](https://github.com/cynthiatreger/az-routing-guide-ep1-vnet-peering-and-virtual-network-gateways#112-standard-hub--spoke-topology-and-non-transitivity-of-vnet-peerings)

[1.2. Connectivity impact of adding a Virtual Network Gateway (ER or VPN)](https://github.com/cynthiatreger/az-routing-guide-ep1-vnet-peering-and-virtual-network-gateways#12-connectivity-impact-of-adding-a-virtual-network-gateway-er-or-vpn)

&emsp;[1.2.1. Azure Virtual Network GW (VPN or ER) for On-Prem connectivity](https://github.com/cynthiatreger/az-routing-guide-ep1-vnet-peering-and-virtual-network-gateways#121azure-virtual-network-gw-vpn-or-er-for-on-prem-connectivity)

&emsp;[1.2.2. On-Prem <=> Spoke VNET propagation](https://github.com/cynthiatreger/az-routing-guide-ep1-vnet-peering-and-virtual-network-gateways#122on-prem--spokes-propagation) ([*Gateway Transit*](https://github.com/cynthiatreger/az-routing-guide-ep1-vnet-peering-and-virtual-network-gateways#1221-gw-transit-scope--the-entire-vnet--the-vnet-range-is-or-is-not-advertised-on-prem) & [*Gateway route propagation*](https://github.com/cynthiatreger/az-routing-guide-ep1-vnet-peering-and-virtual-network-gateways#1222-propagate-gateway-routes-scope--selected-subnets-of-a-vnet-only--the-vnet-range-still-gets-advertised-on-prem) features)
##
# 1.1. Discovery/Reminder of Azure VNET connectivity principles

## 1.1.1. VNET peering & *Effective routes*

To start, we will consider [2 peered VNETs](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview) and a VM in Spoke1 VNET. A single IP range* is allocated to each VNET. In the example below, each IP range is broken into 2 subnets.

* *An Azure Virtual Network (VNET) is made of one or multiple IP ranges and each range is split into subnets.*

<img width="610" alt="image" src="https://user-images.githubusercontent.com/110976272/215267844-9dd42db4-253d-4de9-a9d9-b348e6e37b75.png">

From an Azure networking perspective, according to Spoke1VM's *Effective routes*, Spoke1VM will know about the overall range of its own VNET (but won’t have the granularity of the more specific subnets) and the range of any other peered VNET.

By default all resources in a VNET can communicate outbound to the internet, hence the default route.

## 1.1.2. Standard Hub & Spoke topology and non-Transitivity of VNET peerings

**Peering connections are non-transitive:** in a standard Hub & Spoke topology like depicted below, Spoke1VM will know about the Spoke1 VNET range AND the central VNET (Hub VNET) range, but because there is no direct peering between Spoke1 VNET and Spoke2 VNET, they won’t know about each other. 

Using this peering logic, since the Hub VNET is peered with both Spoke1 and Spoke2 VNETs, the HubTestVM will know about its own Hub VNET range and both Spoke1 & Spoke2 ranges.

<img width="1024" alt="image" src="https://user-images.githubusercontent.com/110976272/215268136-947d048a-c805-4bf5-b637-1ed2db7e563e.png">

# 1.2. Connectivity impact of adding a Virtual Network Gateway (ER or VPN)

## 1.2.1.	Azure Virtual Network GW (VPN or ER) for On-Prem connectivity
Let's look at the **default route propagation** when an Virtual Network GW is deployed (Expressroute GW here, but the results would be similar with a VPN GW). 

The Virtual Network GW is usually hosted in a central Hub VNET, and must be in the *Gatewaysubnet* subnet.

:arrow_right: Whether it is VPN or ER, [if a Virtual Network Gateway (VNG) is added in a Hub VNET](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#border-gateway-protocol) and further connected to On-Prem, the On-Prem  prefixes received by the virtual Network GW will by default automatically be known within this Hub VNET. The Hub VNET range is advertised in return to the On-Prem.

:arrow_right: By default, the On-Prem connectivity is not extended over existing VNET peerings: the On-Prem prefixes received by the Virtual Network GW are NOT readvertised to the peered Spoke VNETs and the Spoke VNET IP ranges are NOT propagated On-Prem.

*The On-Prem is emulated by a VNET connected to the Hub VNET Expressroute circuit.* 

<img width="1024" alt="image" src="https://user-images.githubusercontent.com/110976272/215268623-5e4ca81d-fc4f-49f9-8607-f7c9a986f57d.png">

## 1.2.2.	On-Prem <=> Spoke VNET propagation

### 1.2.2.1. “Gateway transit” scope = the entire VNET & the VNET range is (or is not) advertised On-Prem

[***Gateway Transit***](https://learn.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-peering-gateway-transit?toc=%2Fazure%2Fvirtual-network%2Ftoc.json) allows to extend the scope of the connectivity between Azure and the On-Prem to peered VNETs, as represented on the diagram below for Spoke1 VNET. 

It is a per-VNET peering feature that must be applied on both ends of a peering:

<img width="700" alt="image" src="https://user-images.githubusercontent.com/110976272/215270865-bd19eb5f-1c29-4005-873a-e64b5260e56f.png">

:arrow_right: The On-Prem prefixes will start being “treated” like IP ranges of the Hub VNET: following the VNET peering principle they will get propagated to this directly peered Spoke VNET and be programmed in its VMs, with *Virtual Network Gateway* displayed as Next-Hop type.

:arrow_right: And in return, the Spoke1 VNET range will also be advertised On-Prem.

<img width="1024" alt="image" src="https://user-images.githubusercontent.com/110976272/215283927-34c4d2a5-2e4c-4273-9400-c9f81bec2d25.png">

### 1.2.2.2. “Propagate Gateway Routes” scope = selected subnets of a VNET only & the VNET range still gets advertised On-Prem

Spoke1VM2 is a new VM is added in Spoke1/subnet2. As *GW Transit* is enabled for Spoke1, Spoke1VM2 does know about the On-Prem prefixes. But let's imagine we want the resources in this specific subnet to be isolated from On-Prem. 

:arrow_right: If a route table with [***Gateway route propagation*** = disabled](https://learn.microsoft.com/en-us/azure/virtual-network/manage-route-table#create-a-route-table:~:text=Propagate%20gateway%20routes,to%20Disabled.) is applied to a specific subnet, either in the VNET hosting the Virtual Network Gateway or in a peered VNET, then the On-Prem prefixes received by the GW will NOT be propagated to this given subnet.

:arrow_right: When *GW route propagation* is disabled, even on all the subnets of a VNET, the overall VNET IP range is STILL propagated On-Prem.

<img width="1118" alt="image" src="https://user-images.githubusercontent.com/110976272/215284125-96dceab0-f9c2-438e-b24b-8fd5e414ad88.png">

##
### [>> EPISODE #2](https://github.com/cynthiatreger/az-routing-guide-ep2-nic-routing)


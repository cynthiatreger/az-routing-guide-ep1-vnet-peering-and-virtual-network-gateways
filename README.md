# EPISODE #1: Discovery/Reminder of Azure VNET Connectivity & Impact of Virtual Network Gateways

[1.1. Discovery/Reminder of Azure VNET connectivity principles](https://github.com/cynthiatreger/az-routing-guide-part1-vnet-peering-and-virtual-network-gateways/edit/main/README.md#11-discoveryreminder-of-azure-vnet-connectivity-principles)

[1.1.1. VNET peering & Effective routes](https://github.com/cynthiatreger/az-routing-guide-ep1-vnet-peering-and-virtual-network-gateways/edit/main/README.md#111-vnet-peering--effective-routes)
  
[1.1.2. Standard H&S topology and non-transitivity of VNET peerings](https://github.com/cynthiatreger/az-routing-guide-ep1-vnet-peering-and-virtual-network-gateways/edit/main/README.md#112standard-hs-topology-and-non-transitivity-of-vnet-peerings)

[1.2. Connectivity Impact of adding a Virtual Network Gateway (ER or VPN)](https://github.com/cynthiatreger/az-routing-guide-part1-vnet-peering-and-virtual-network-gateways/edit/main/README.md#12-connectivity-impact-of-adding-a-virtual-network-gateway)

## 1.1. Discovery/Reminder of Azure VNET connectivity principles

### 1.1.1. VNET peering & Effective routes

To start, we will consider [2 peered VNETs](https://github.com/cynthiatreger/az-routing-guide-ep1-vnet-peering-and-virtual-network-gateways/edit/main/README.md#111-vnet-peering--effective-routes) and a VM in Spoke1 VNET. A single IP range* is allocated to each VNET. In the example below, each IP range is broken into 2 subnets.

(* an Azure Virtual Network (VNET) is made of one or multiple IP ranges and each range is split into subnets.)

<img width="632" alt="image" src="https://user-images.githubusercontent.com/110976272/215120121-7a8b2ddc-b5f7-4939-bae3-8080725b90d2.png">

From an Azure networking perspective, according to Spoke1VM's *Effective routes*, Spoke1VM will know about the overall range of its own VNET (but won’t have the granularity of the more specific subnets) and the range of any other peered VNET.

By default all resources in a VNET can communicate outbound to the internet, hence the default route.


### 1.1.2. Standard Hub & Spoke topology and non-transitivity of VNET peerings

**Peering connections are non-transitive.** 

In a standard Hub & Spoke topology like depicted below, Spoke1VM will know about the Spoke1 VNET range AND the central VNET (Hub VNET) range, but because there is no direct peering between Spoke1 VNET and Spoke2 VNET, they won’t know about each other. 
Using this peering logic, since the Hub VNET is peered with both Spoke1 and Spoke2 VNETs, the HubVM will know about its own Hub VNET range and both Spoke1 & Spoke2 ranges.
 
Spoke1VM’s Effective routes
 
HubVM’s Effective routes
 


## 1.2. Connectivity Impact of adding a Virtual Network Gateway (ER or VPN)

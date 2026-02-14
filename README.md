# Enterprise-Network

## Project Overview

This project simulates a highly available, scalable enterprise network for a multi-site company. Designed using the Cisco Three-Tier Hierarchical Model (Core, Distribution, Access), the topology connects a centralized Data Center with two remote office branches (Office A & B) and an ISP uplink.

The network is designed for redundancy, performance, and security, utilizing Layer 3 switching in the core and distribution layers, with a full suite of network services (DNS, DHCP, FTP, etc.).

## Topology

<img width="2658" height="1528" alt="image" src="https://github.com/user-attachments/assets/8a550cff-b019-4813-af01-7ded06230fba" />

---

## Key Features & Technologies

**Hierarchical Design**: 
- Implemented Core, Distribution, and Access layers to ensure scalability and fault isolation.

**Redundancy & High Availability:**
  - [**EtherChannel (LACP):**](https://github.com/Alan1234111/Enterprise-Network/blob/main/documentation/01_Redundancy_HA/Etherchannel.md) Bundled links between Core and Distribution switches for increased bandwidth and failover.
  - [**HSRP:**](https://github.com/Alan1234111/Enterprise-Network/blob/main/documentation/01_Redundancy_HA/HSRP.md)  Configured First Hop Redundancy Protocols for gateway redundancy at the Distribution layer.
  - [**RapidPVST:**](https://github.com/Alan1234111/Enterprise-Network/blob/main/documentation/01_Redundancy_HA/RapidPVST.md) Rapid Per Vlan Spanning Tree tuning to prevent loops while maintaining redundant paths.

**Routing & Switching:**
 - [**VLANs:**](https://github.com/Alan1234111/Enterprise-Network/blob/main/documentation/02_Routing_Switching/VLANs.md) Segmentation of Voice, Data, Management, and Server traffic.
 - [**Inter-VLAN Routing:**](https://github.com/Alan1234111/Enterprise-Network/blob/main/documentation/02_Routing_Switching/Inter-VLAN%20Routing.md) Configured on Multilayer Switches.
 - [**Dynamic Routing (OSPF):**](https://github.com/Alan1234111/Enterprise-Network/blob/main/documentation/02_Routing_Switching/Dynamic%20Routing%20(OSPF).md) Implemented for automatic route exchange between the Core, Distribution and Edge Router.

**Wireless:**
 - **[Centralized Wireless (WLC)](https://github.com/Alan1234111/Enterprise-Network/blob/main/documentation/03_Wireless/Wireless.md):** Lightweight Access Points (LWAP) managed by a Wireless LAN Controller (WLC 3504).

**Services:**
- Full server stack implementation: FTP, DHCP, Syslog, Email, Web, TFTP, and NTP.
- NAT/PAT configured on the Edge Router for Internet access.

## Hardware & Bill of Materials
**Core & WAN Layer:**

2x Cisco 3650-24PS Multilayer Switches (Core Backbone)

1x Cisco 2911 ISR Router (Enterprise Edge)

1x Cisco 2911 ISR Router (ISP Simulation)

**Distribution & Access Layer:**

6x Cisco 3650-24PS Multilayer Switches (Distribution Layer)

8x Cisco 2960-24TT Switches (Access Layer)

**End Devices & Wireless:**

1x Cisco 3504 Wireless LAN Controller (WLC)

2x Lightweight Access Points (LWAP)

Endpoint Mix: PCs, Laptops, and Cisco IP Phones (VoIP)

**Server Farm (Data Center):**

Infrastructure: DHCP, DNS (Google 8.8.8.8), NTP, Syslog, TFTP

Application: Web Server (HTTP), Email Server (SMTP/POP3), FTP Server

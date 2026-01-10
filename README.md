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
  - **EtherChannel (LACP):** Bundled links between Core and Distribution switches for increased bandwidth and failover.
  - **HSRP/VRRP:**  Configured First Hop Redundancy Protocols for gateway redundancy at the Distribution layer.
  - **RapidPVST:** Rapid Per Vlan Spanning Tree tuning to prevent loops while maintaining redundant paths.

**Routing & Switching:**
 - **VLANs:** Segmentation of Voice, Data, Management, and Server traffic.
 - **Inter-VLAN Routing:** Configured on Multilayer Switches.
 - **Dynamic Routing (OSPF):** Implemented for automatic route exchange between the Core, Distribution and Edge Router.

**Wireless:**
 - Lightweight Access Points (LWAP) managed by a Wireless LAN Controller (WLC 3504).

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

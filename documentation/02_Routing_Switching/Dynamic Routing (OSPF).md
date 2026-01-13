# Dynamic Routing (OSPF)

## 1. Overview

To automate the exchange of routing information between the Data Center, Office Branches, and the Edge Router, I deployed Multi-Area OSPF (Open Shortest Path First).

### Why I used it:

* **Scalability:** Unlike static routing, OSPF automatically adapts to network changes. If a new branch is added, the network learns about it without manual configuration on every device.

* **Vendor Neutral:** As an open standard, it allows future integration of non-Cisco equipment.

* **Fast Convergence:** Using the Shortest Path First (Dijkstra) algorithm, the network quickly recalculates routes if a link fails.

* **Load Balancing:** OSPF supports ECMP (Equal-Cost Multi-Path), allowing traffic to be split across redundant links between Distribution and Core layers.

### Why Multi-Area?

* **Fault Isolation:** If a link flaps in the Office network (Area 10), it triggers an SPF recalculation only within Area 10. The Backbone (Area 0) is unaffected, receiving only a summary update.

* **Reduced Overhead:** Routers in one area do not need full topological details of the other area, saving CPU and memory resources.

## 2. Implementation Details

**Router Roles:**

* **Internal Routers:** Switches in the Core layer (operating purely in area 0)
* **ABR (Area Border Routers):** Distribution Switches, connected to the area 0 (backbone) and also area 10 (internal routes)
* **ASBR (Autonomous System Boundary Router):** The Edge Router `R1` acts as the ASBR, injecting the external default route into the OSPF domain.
* 

**Configuration Snippet:**

### DSW-A1 (ABR)

```cisco
interface range g1/1/1-2
ip ospf network point-to-point
exit
router ospf 1
network 10.0.0.0 0.0.0.255 area 0
passive-interface l0
passive-interface vlan 10
passive-interface vlan 20
passive-interface vlan 50
passive-interface vlan 100

network 10.0.10.0 0.0.0.255 area 10
network 10.0.20.0 0.0.0.255 area 10
network 10.0.100.0 0.0.0.255 area 10
network 10.0.50.0 0.0.0.255 area 10

auto-cost reference-bandwidth 1000000
```

### R1 (ASBR)

```cisco
ip route 0.0.0.0 0.0.0.0 203.0.113.2

interface range g0/0-1
ip ospf network point-to-point

router ospf 1
network 10.0.0.48 0.0.0.3 area 0
network 10.0.0.52 0.0.0.3 area 0
network 10.0.0.108 0.0.0.0 area 0

passive-interface l0
default-information originate

auto-cost reference-bandwidth 1000000
```

### CSW1 (ABR)

```cisco
interface range g1/1/1-4,g1/0/4-5,g1/0/1
ip ospf network point-to-point

router ospf 1
network 10.0.0.48 0.0.0.3 area 0
network 10.0.0.56 0.0.0.3 area 0
network 10.0.0.60 0.0.0.3 area 0
network 10.0.0.64 0.0.0.3 area 0
network 10.0.0.68 0.0.0.3 area 0
network 10.0.0.72 0.0.0.3 area 0
network 10.0.0.92 0.0.0.3 area 0
network 10.0.0.96 0.0.0.3 area 0
network 10.0.0.109 0.0.0.0 area 0

passive-interface l0
auto-cost reference-bandwidth 1000000

```

## 3. Verification
To verify the topology and role assignments, I utilized specific OSPF commands.

---

**Check 1:** Verifying ABR Role On the Distribution Switch.

**Command:** 'show ip ospf 1'

**Output (Example from DSW-A1):**

<img width="1137" height="1058" alt="image" src="https://github.com/user-attachments/assets/6ec56c98-80c1-4a82-8bcb-0332a0f472b6" />

---

**Check 2:**  Verifying that the Core Switch learns about internal VLANs as "Inter-Area" routes.

**Command:** 'show ip route ospf'

<img width="1190" height="181" alt="image" src="https://github.com/user-attachments/assets/7b4b5f37-46a0-4a14-ae9c-a3ab20d56f60" />

* **O IA:** The Core sees VLAN 10/20 as coming from a different area (Area 10), confirming the ABR boundary is working.

---

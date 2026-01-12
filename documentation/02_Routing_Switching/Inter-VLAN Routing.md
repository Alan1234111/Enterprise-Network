# Inter-VLAN Routing (Layer 3 Switching)
## 1. Overview

In this architecture, I shifted the routing responsibility from the Edge Router to the Multilayer Switches (Distribution Layer). This design implements Inter-VLAN Routing using Switched Virtual Interfaces (SVIs)

### Why I used it:

* **Performance:** Traffic is routed at near wire-speed by the switch hardware (ASICs) rather than relying on the router's CPU.

* **Bandwidth Efficiency:** Eliminates the "Router-on-a-Stick" bottleneck where traffic must go up to the router and back down just to reach a different VLAN.

* **Scalability:** The Edge Router is freed up to handle WAN traffic, NAT, and Firewalling, while internal routing is handled by the high-speed backplane of the switches. Configured trunk links between switches to carry traffic for all VLANs using standard dot1q tagging.


## 2. Implementation Details

I created SVIs (Switched Virtual Interfaces) for each VLAN on the Distribution Switches (DSW). These virtual interfaces act as the default gateways for the subnets (in conjunction with HSRP).

### **Routing Logic:**

Internal Traffic: Traffic moving between VLANs (e.g., HR to IT) is routed locally on the Distribution Switch.

External Traffic: Traffic destined for the Internet is routed from the Distribution Switch up to the Core, and finally to the Edge Router via OSPF.


**Configuration Snippet:**

**1. Enabling IP Routing (Global):** Crucial step to convert the switch from Layer 2 to Layer 3 mode.

```cisco
ip routing
```

**2. Configuring SVIs:** Below is the configuration for the routing interfaces. Note that these IPs match the HSRP physical interface configuration.

### DSW-A1

```cisco
! Interface for HR Department
interface Vlan10
 ip address 10.0.10.2 255.255.255.0
 no shutdown

! Interface for IT Department
interface Vlan20
 ip address 10.0.20.2 255.255.255.0
 no shutdown

[...]
```

**3. Routed Port to Core (Uplink):** Instead of using SVIs for the uplink, I used a physical Routed Port to connect to the Core layer

```cisco
interface g1/1/1
no switchport
ip address 10.0.0.62 255.255.255.252
no shutdown

interface g1/1/2
no switchport
ip address 10.0.0.78 255.255.255.252
no shutdown
```

Trunking & Native VLAN
To enhance security, I changed the Native VLAN from the default (1) to an unused VLAN (1000) on all trunk links. This mitigates VLAN Hopping attacks.

Configuration on Uplinks:

### ASW-A2
```cisco
interface range g0/1-2
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,50,99,100
```


## 3. Verification
To verify that the switch is actively routing traffic, I checked the routing table

**Command:** 'show ip route'

**Output (Example from DSW-A1):**

```cisco
Gateway of last resort is 10.0.0.77 to network 0.0.0.0

     10.0.0.0/8 is variably subnetted, 44 subnets, 4 masks
C       10.0.0.0/28 is directly connected, Vlan99     <-- Local (Management)
L       10.0.0.2/32 is directly connected, Vlan99
...
C       10.0.10.0/24 is directly connected, Vlan10    <-- Local (HR)
L       10.0.10.2/32 is directly connected, Vlan10
C       10.0.20.0/24 is directly connected, Vlan20    <-- Local (IT)
L       10.0.20.2/32 is directly connected, Vlan20
...
C       10.0.50.0/24 is directly connected, Vlan50    <-- Local (Wi-Fi)
L       10.0.50.2/32 is directly connected, Vlan50
...
C       10.0.100.0/24 is directly connected, Vlan100  <-- Local (Phones A)
L       10.0.100.2/32 is directly connected, Vlan100
...
O*E2 0.0.0.0/0 [110/1] via 10.0.0.77, 00:00:24, GigabitEthernet1/1/2
```

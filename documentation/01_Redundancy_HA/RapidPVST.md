# Rapid PVST Implementation 

## 1. Overview

In this network, I upgraded the default Spanning Tree Protocol to Rapid PVST+ (Per-VLAN Spanning Tree Plus). This ensures loop-free Layer 2 topology while providing significantly faster convergence times compared to classic 802.1D.

### Why I used it:

* **Rapid Convergence:** Reduces link recovery time from ~30-50 seconds (Standard STP) to under a second in most cases (802.1w), which is critical for VoIP and DHCP services.

* **VLAN Awareness:** Creates a separate Spanning Tree instance for each VLAN. This allows for load balancing where traffic for VLAN 10 uses Link A, while traffic for VLAN 20 uses Link B.

* **Loop Prevention:** Automatically blocks redundant paths to prevent broadcast storms, while keeping them available as backups

## 2. Implementation Details

**Root Bridge Strategy:** To ensure optimal traffic flow, I manually configured the Distribution Switches (DSW) to act as Root Bridges, preventing Access Switches from ever becoming the root.

**Primary Root:** `DSW-A1` (For VLANs 10, 20, 99), `DSW-A2` (For VLANs 50, 100)

**Secondary Root:** `DSW-A1` (For VLANs 50, 100),  `DSW-A2` (For VLANs 10, 20, 99)

Note: This alignment matches the HSRP Active/Standby configuration to ensure Layer 2 and Layer 3 paths are congruent.


**Configuration Snippet:**
Below is the configuration for VLAN 10 (HR Dept). Note that DSW-A1 is configured as the Active Router.

Access Layer Optimizations: On all Access Switches (ASW), ports connected to end devices (PCs, Phones, WLC) are configured as Edge Ports (Portfast).

---
1. Enabling Rapid PVST+ (Global Config):

```cisco
spanning-tree mode rapid-pvst
```

2. Configuring Root Priorities (Distribution Layer): Configuration on DSW-A1 to make it the Primary Root for VLAN 10:

```cisco
spanning-tree vlan 10 priority 0
```

Configuration on DSW-A2 to make it the Secondary Root for VLAN 10

```cisco
spanning-tree vlan 10 priority 4096
```

3. PortFast & BPDU Guard (Access Layer): Configuration on ASW-A1 interfaces connected to PCs:
```cisco
interface range FastEthernet0/1-24
 spanning-tree portfast
 spanning-tree bpduguard enable
```


## 3. Verification
To verify the Spanning Tree state and Root Bridge location:

**Command:** 'show spanning-tree vlan 10'

**Output (Example from DSW-A1):**

<img width="1252" height="631" alt="image" src="https://github.com/user-attachments/assets/b5319c35-aea2-41da-a741-b53fa71da4bf" />

* **This bridge is the root:** Confirms correct placement of the Root Bridge.

* **Type: P2p:** Confirming Rapid Spanning Tree is operating on full-duplex links.

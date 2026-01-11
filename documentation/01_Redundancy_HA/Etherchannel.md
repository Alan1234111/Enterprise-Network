# EtherChannel Implementation (LACP)

## 1. Overview

In this network, I implemented EtherChannel to bundle multiple physical links between the Core Switches, as well as between the Distribution Switches, into single logical links.

### Why I used it:
* **Bandwidth Aggregation:** Combines the speed of two Gigabit links into a single 2Gbps pipe, reducing bottlenecks at the core.
* **Redundancy:** If one cable fails, the link stays up, and traffic continues to flow over the remaining cable without triggering Spanning Tree reconvergence.
* **Protocol:** I utilized **LACP (Link Aggregation Control Protocol)** as it is the industry standard (IEEE 802.3ad) compatible with non-Cisco gear, rather than the proprietary PAgP.

## 2. Implementation Details

**Location:**
* Between `CSW1` and `CSW2` (L3)
* Between `DSW-A1` and `DSW-A2` (L2)
* Between `DSW-B1` and `DSW-B2` (L2)
* Between `DSW-S1` and `DSW-S2` (L2)

**Configuration Snippet:**
Below is the configuration used on the Core Switch to create the bundle.

```cisco
! Interface Configuration
interface range g1/0/2-3
shutdown
no switchport
channel-protocol lacp
channel-group 1 mode active

! Port-Channel Interface
interface po1
ip address 10.0.0.57 255.255.255.252
```


## 3. Verification
To verify the bundle is operational, I used the following checks:

**Command:** 'show etherchannel summary'

**Output:**

<img width="936" height="548" alt="image" src="https://github.com/user-attachments/assets/633e48a8-b225-4ca3-97ca-9bff234c2063" />

* **RU:** Layer 3 / In Use (Success)
* **P:** Bundled in Port-channel (Success)



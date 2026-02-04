# HSRP Implementation 

## 1. Overview

In this network, I implemented HSRP (Hot Standby Router Protocol) on the Distribution Layer switches. This ensures high availability for the default gateway services provided to end devices (PCs, Phones, Laptops).

### Why I used it:

* **Gateway Redundancy:** Provides a seamless failover mechanism. If the primary Distribution Switch (Active) fails, the backup switch (Standby) immediately takes over the routing responsibilities.

* **Transparency:** End devices are configured with a single Virtual IP (VIP) as their default gateway. They remain unaware of physical router failures.

* **Load Balancing (VLAN-based):** I configured HSRP groups so that DSW-A1 is Active for half the VLANs, while DSW-A2 is Active for the other half, utilizing both switches simultaneously under normal conditions.

## 2. Implementation Details

**Location:**
* Between `DSW-A1` Active: (VLAN: 10, 20, 99) and `DSW-A2` Standby: (VLAN: 50, 100)
* Between `DSW-B1` Active: (VLAN: 30, 40) and `DSW-B2` Standby: (VLAN: 99, 100)
* Between `DSW-S1` Active: (VLAN: 60, 99) and `DSW-S2` Standby: (VLAN: 70)

**Configuration Snippet:**
Below is the configuration for VLAN 10 (HR Dept). Note that DSW-A1 is configured as the Active Router.

### DSW-A1
```cisco
interface vlan 10
ip address 10.0.10.2 255.255.255.0
no shutdown
standby version 2
standby 10 ip 10.0.10.1
standby 10 priority 105
standby 10 preempt
```

### DSW-A2
```cisco
interface vlan 10
ip address 10.0.10.3 255.255.255.0
no shutdown
standby version 2
standby 10 ip 10.0.10.1
standby 10 preempt
```


## 3. Verification
To verify the HSRP state and ensure the correct switch is acting as the Gateway, I used the following checks:

**Command:** 'show standby brief'

**Output (Example from DSW-A1):**

<img width="970" height="219" alt="image" src="https://github.com/user-attachments/assets/8cb7f9cd-63cb-4292-9014-3b9681695c79" />


* Active: Indicates this switch is currently routing traffic for the VLAN.

* Standby: Indicates this switch is waiting to take over if the Active peer fails.

* Virtual IP: The address configured on all PCs as the Default Gateway.

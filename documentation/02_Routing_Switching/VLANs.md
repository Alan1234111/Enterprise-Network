# VLANs Implementation 

## 1. Overview

In this project, I implemented VLANs (Virtual Local Area Networks) to segment network traffic logically rather than physically. This design improves security, reduces broadcast domain size, and organizes the network structure by department.

### Why I used it:

* **Segmentation:** Departments (HR, IT, Sales) are isolated into their own subnets.

* **Voice VLAN:** I implemented a dedicated Voice VLAN to ensure VoIP traffic is separated from data traffic, allowing for future QoS implementation.

* **Management VLAN:** Following security best practices, I moved switch management traffic off the default VLAN 1 to a dedicated secured VLAN.

* **802.1Q Trunks:** Configured trunk links between switches to carry traffic for all VLANs using standard dot1q tagging.


## 2. Implementation Details

### VLAN Schema

<img width="901" height="1219" alt="image" src="https://github.com/user-attachments/assets/28c6419a-bdf2-46c4-be9c-cff2f9ee9fdb" />


**Configuration Snippet:**
For ports connected to Cisco IP Phones with PCs attached behind them, i implemented Access VLAN + Voice VLAN

### ASW-A2

```cisco
interface f0/1
switchport mode access
switchport access vlan 10
switchport voice vlan 100
spanning-tree portfast
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
To verify the VLAN assignment and Trunk status:

**Command:** 'show vlan brief'

**Output (Example from ASW-A2):**

<img width="1240" height="568" alt="image" src="https://github.com/user-attachments/assets/9f0ac392-5fc1-464b-bef7-5697ae9d5cc8" />

---

**Command:** 'show interfaces trunk'

**Output (Example from ASW-A2):**

<img width="1160" height="481" alt="image" src="https://github.com/user-attachments/assets/94be1c78-57a8-4615-8639-553ca234bad9" />





# Layer 2 Security Hardening

## 1. Overview
In this project, I implemented robust Layer 2 security features to mitigate common LAN attacks such as MAC Flooding, DHCP Starvation, Rogue DHCP Servers, and ARP Spoofing (Man-in-the-Middle).

### Security Features Implemented:
* **Port Security:** Restricts input to an interface by limiting and identifying MAC addresses of the stations allowed to access the port.
  
* **DHCP Snooping:** Filters untrusted DHCP messages and builds a binding database of valid hosts.
  
* **Dynamic ARP Inspection (DAI):** Validates ARP packets in the network, discarding malicious ARP packets used in poisoning attacks.

* **BPDU Guard:** protect the Spanning Tree topology by automatically disabling access ports if they receive BPDU packets (indicating a rogue switch connection).

## 2. Implementation Details


### A. Port Security 
I configured Port Security on all edge ports connected to end-user devices.

**Policy:**
* **Maximum MACs:** 2 (To allow for a PC + IP Phone) / 1 (In the data center connected to a server)
  
* **Violation Mode:** `Restrict` (Drops packets from invalid MACs and logs the event via Syslog/SNMP, but keeps the port UP).
  
* **Aging:** Static sticky MAC addresses.

**Configuration Snippet (Access Switch):**

```cisco
switchport port-security
switchport port-security maximum 2
switchport port-security violation restrict
switchport port-security mac-address sticky
```
---

### B. DHCP Snooping
This feature acts as a firewall between untrusted hosts and the DHCP servers.

**Logic:**

* **Trusted Ports:** Uplinks to Distribution Switches and the link to DHCP Server

* **Untrusted Ports:** All edge ports connected to PCs/Phones.

**Configuration Snippet (Access Switch):**

```cisco
ip dhcp snooping
ip dhcp snooping vlan 10,20,50,99,100
no ip dhcp snooping information option 

!WLC
interface f0/2  
ip dhcp snooping trust
ip dhcp snooping limit rate 100

interface f0/1
ip dhcp snooping limit rate 15

interface range g0/1-2 
ip dhcp snooping trust
```
---

### C. Dynamic ARP Inspection (DAI)
DAI relies on the database built by DHCP Snooping. It intercepts all ARP requests and replies on untrusted ports and verifies that the IP-to-MAC mapping matches the valid DHCP binding.

**Configuration Snippet (Access Switch):**

```cisco
ip arp inspection vlan 10,20,50,99,100
ip arp inspection validate dst-mac ip src-mac

interface range g0/1-2 
ip arp inspection trust
```
---

**Note on Data Center:** Server VLANs (60, 70) are excluded from DHCP Snooping and DAI due to static IP addressing. I secured these segments using Port Security and L3 ACLs instead.


## 3. Verification
Checking for violations on access ports.

### A. Port Security

**Note on Maximum Address:** Due to a specific behavior in Cisco Packet Tracer where IP Phones momentarily present multiple MAC addresses during initialization, I increased the maximum secure MAC address count to prevent false positive violations during device boot-up.


**Command:** `show port-security interface f0/1`

<img width="456" height="201" alt="image" src="https://github.com/user-attachments/assets/b450a58e-0ec9-4a79-b7ba-1ff193258ea9" />

---

### B. DHCP Snooping

To ensure the security features are active and the binding database is being populated:

**Command:** `show ip dhcp snooping binding`

<img width="701" height="93" alt="image" src="https://github.com/user-attachments/assets/a68ec6d9-44d3-48ef-8b2c-a2593d53b557" />

---

### C. Dynamic ARP Inspection (DAI)

To ensure everything is configured properly and active on the correct VLANs:

**Command:** `show ip arp inspection`

<img width="578" height="202" alt="image" src="https://github.com/user-attachments/assets/9124f51e-3b56-4e30-9cd6-baa9b6b74923" />





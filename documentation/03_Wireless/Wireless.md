# Wireless LAN Implementation (WLC & CAPWAP)

## 1. Overview

To provide secure and scalable mobility, I implemented a Centralized Wireless Architecture using a Cisco Wireless LAN Controller (WLC 3504) and Lightweight Access Points (LWAPs).

### Why I used it:
* **Split-MAC Architecture:** Management and security functions are centralized on the WLC, while real-time functions (beacons, encryption) are handled by the APs.
* **CAPWAP Tunneling:** All client traffic is encapsulated in CAPWAP tunnels between the AP and the WLC. This allows for seamless roaming without changing IP addresses (Layer 3 Roaming).

## 2. Implementation Details

---

### **A. Switch Port Configuration:**
**WLC Uplink (Trunk):**
```cisco
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 99
switchport trunk allowed vlan 50,99
```

**AP Connection (Access Port):**
```cisco
interface f0/1
switchport mode access
switchport access vlan 99 ! AP Management VLAN
spanning-tree portfast
```

---

### **B. WLC Configuration (Logical):**


**Dynamic Interface Configuration:**

<img width="1202" height="748" alt="image" src="https://github.com/user-attachments/assets/ef5e8c0c-6f6b-417c-ac8a-a5bb9b038416" />


**WLANs Configuration:**

<img width="1189" height="517" alt="image" src="https://github.com/user-attachments/assets/dc5eeff4-8b48-43da-aec2-e125a075c4d9" />

---

<img width="1182" height="509" alt="image" src="https://github.com/user-attachments/assets/0c20ce89-72b9-4fce-81ae-92423145c32d" />


### **C. End Point Configuration:**

<img width="834" height="398" alt="image" src="https://github.com/user-attachments/assets/68a936f1-c642-4ef1-8349-6b3b3c2133cf" />



## 3. Verification
To verify the wireless operations, I checked the AP registration status and client connectivity.

<img width="806" height="311" alt="image" src="https://github.com/user-attachments/assets/ebf99ba2-3a8a-4606-99c4-79e479df7cfa" />

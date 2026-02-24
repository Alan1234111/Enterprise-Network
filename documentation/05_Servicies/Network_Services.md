# Network Services & Edge NAT

## 1. Overview

I implemented a comprehensive suite of centralized network services in the Data Center, alongside Network Address Translation (NAT) on the Edge Router for secure Internet access.

### Services Deployed:

* **Public Services:** (VLAN 60): Web, Email.

* **Internal Services:** (VLAN 70): FTP, TFTP, DHCP, NTP, SYSLOG.

### Why I used it:

* **Centralized Management:** Grouping servers in dedicated Data Center VLANs allows for strict, granular security policies (Zero-Trust ACLs) rather than having services scattered across the network.

* **Scalable IP Allocation:** A centralized DHCP server handles scopes for multiple VLANs.

* **Network Logs & Accuracy:** Implementing a central Syslog server paired with an NTP server ensures that all security events and interface state changes are logged accurately with synchronized timestamps across the entire topology.

* **IPv4 Conservation (NAT/PAT):** Using Port Address Translation (NAT Overload) allows the entire enterprise network (using private RFC 1918 subnets) to share a single public IP address provided by the ISP.

* **Disaster Recovery:** Utilizing centralized TFTP and FTP servers enables secure, centralized backups of router/switch configurations and IOS images.

## 2. Implementation Details

While the server parameters (Email accounts, Web files) were configured directly on the Data Center servers, the network required specific routing and relay configurations to support them.

### A. DHCP IP Helper

```cisco
! Applying DHCP Relay to the HR Department SVI

int vlan 10
ip helper-address 10.0.70.6
```

### B. NAT Overload (PAT) on Edge Router

```cisco
! Applying PAT on the Edge Router

access-list 1 permit 10.0.10.0 0.0.0.255
access-list 1 permit 10.0.20.0 0.0.0.255
access-list 1 permit 10.0.100.0 0.0.0.255
access-list 1 permit 10.0.30.0 0.0.0.255
access-list 1 permit 10.0.40.0 0.0.0.255
access-list 1 permit 10.0.101.0 0.0.0.255
access-list 1 permit 10.0.60.0 0.0.0.255

int g0/0/0
ip nat outside

int range g0/0-1
ip nat inside

ip nat inside source list 1 interface g0/0/0 overload
```

### C. SYSLOG

```cisco
logging host 10.0.70.8
```

### D. SSH

```cisco
ip domain-name alancompany
crypto key generate rsa
4096
ip ssh version 2
line vty 0 15
transport input ssh
login local
logging synchronous
```


## 3. Verification

To ensure all network services and edge translations are operating correctly across the enterprise, I performed a series of functionality tests.

### DHCP

**Objective:** Verify that end-user devices successfully receive IP addressing dynamically from the centralized Data Center server via the IP Helper relay.

<img width="1608" height="1350" alt="image" src="https://github.com/user-attachments/assets/2dd49cdc-f9ac-40f3-b414-62b85fd1447d" />

---

<img width="1966" height="528" alt="image" src="https://github.com/user-attachments/assets/0e0c1fdf-7545-4a92-8d14-8843a023f838" />

### NAT Overload

**Objective:** Confirm that internal private IP addresses (RFC 1918) are successfully translated to the single public IP address on the Edge Router when accessing external networks.

<img width="1357" height="609" alt="image" src="https://github.com/user-attachments/assets/54e7aca9-458b-41ce-8f2c-46f4149ced40" />


### SYSLOG

**Objective:** Ensure that infrastructure devices are successfully sending log messages (like interface state changes or security events)

<img width="2826" height="427" alt="image" src="https://github.com/user-attachments/assets/55f27d19-bc11-4bc8-a4e0-407f2c429e5b" />

### SSH

**Objective:** Test secure remote access to network devices using SSHv2

<img width="1895" height="739" alt="image" src="https://github.com/user-attachments/assets/4cdb8e85-1301-4ec7-8aa6-50b17745f691" />

### WEB & DNS

**Objective:** Verify that the internal DNS server successfully resolves corporate hostnames to the correct IP addresses, and that the Web server correctly delivers HTTP/HTTPS content to end-users

<img width="2003" height="858" alt="image" src="https://github.com/user-attachments/assets/7b23e729-94e4-400d-9b2b-240443942433" />

### EMAIL

**Objective:** Ensure that email clients can successfully communicate with the centralized Email server to send (SMTP) and receive (POP3) internal corporate communications.

<img width="1636" height="683" alt="image" src="https://github.com/user-attachments/assets/f9694657-5b0a-466d-bfdf-e5878db42016" />

---

<img width="2006" height="972" alt="image" src="https://github.com/user-attachments/assets/71614ba1-c5c8-44db-9593-c0619d6dd75d" />

### FTP

**Objective:** Validate that authorized users can successfully authenticate and transfer files to the central FTP server located in the Data Center.

<img width="1583" height="901" alt="image" src="https://github.com/user-attachments/assets/198061d2-9068-4fcc-8230-2ccd706f3052" />





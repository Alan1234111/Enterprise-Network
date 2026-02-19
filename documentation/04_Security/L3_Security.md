# Layer 3 Security: Access Control Lists (ACLs)

## 1. Overview
I implemented Access Control Lists (ACLs) at the Layer 3 boundaries to filter traffic between departments, secure infrastructure, and restrict management access.

### Security Features Implemented:

* **Standard ACLs:** Used primarily for restricting VTY (SSH/Telnet) administrative access and identifying internal traffic for NAT overload.

* **Extended ACLs:** Used for granular traffic filtering based on Source IP, Destination IP, Protocol (TCP/UDP), and Port Number to enforce departmental segmentation.

## 2. Implementation Details

### A. Management Plane Protection (SSH)
To prevent unauthorized administrative access, I restricted the VTY lines on all routers and switches so that only the IT Department (VLAN 20) can initiate remote SSH connections.

**Policy:** 

* **Permitted:** IT_DEPT Subnet (10.0.20.0/24)

* **Denied:** All other subnets (Implicit Deny)

* **Application:** Inbound on VTY lines 0-15.

**Configuration Snippet (Applied globally on devices):**

```cisco
access-list 10 permit 10.0.20.0 0.0.0.255 
access-list 10 deny any

line vty 0 15
access-class 10 in
```

---

### B. Internal Segmentation (Extended ACL)

I implemented an Extended ACL to enforce security policies between departments.

#### 1. **Policy Logic (HR_ACL):**

* **Permitted:** Explicit access is granted to necessary business services like the Corporate Web Server (HTTP/HTTPS), Email (SMTP/POP3), and time synchronization (NTP)

* **Denied:** HR is prevented from reaching the Sales Department subnet, as well as infrastructure servers (TFTP, Syslog, FTP)

* **Application:** Applied Inbound on the HR VLAN (SVI 10) on the Distribution Switches in Office A.

```cisco
ip access-list extended HR_ACL
 ! 1. Deny Traffic to Sales Department
 deny ip 10.0.10.0 0.0.0.255 10.0.30.0 0.0.0.255   

 ! 2. Explicitly Permit Business Services (Data Center)
 permit tcp 10.0.10.0 0.0.0.255 host 10.0.60.4 eq 80    ! WWW Server (HTTP)
 permit tcp 10.0.10.0 0.0.0.255 host 10.0.60.4 eq 443   ! WWW Server (HTTPS)
 permit tcp 10.0.10.0 0.0.0.255 host 10.0.60.5 eq 25    ! Email Server (SMTP)
 permit tcp 10.0.10.0 0.0.0.255 host 10.0.60.5 eq 110   ! Email Server (POP3)
 permit udp 10.0.10.0 0.0.0.255 host 10.0.70.7 eq 123   ! NTP Server

 ! 3. Deny Infrastructure / Management Servers
 deny udp 10.0.10.0 0.0.0.255 host 10.0.70.5            ! TFTP Server
 deny udp 10.0.10.0 0.0.0.255 host 10.0.70.8            ! Syslog Server
 deny tcp 10.0.10.0 0.0.0.255 host 10.0.70.4            ! FTP Server

 ! 4. Permit all other traffic (e.g., Internet Access)
 permit ip any any

! Apply to Interface
interface Vlan10
 ip access-group HR_ACL in
```

#### 2. **Policy Logic (IT_ACL):**

* **Permitted:**
  - Full Access to Data Center
  - Explicit permission to use SSH (TCP 22) to manage network infrastructure in the Management VLAN (10.0.0.0/24)
  - Full access to other user subnets (Sales, HR, etc.) for remote support.

* **Denied:** -

* **Application:** Applied Inbound on the IT VLAN (SVI 20) on the Distribution Switches in Office A.

```cisco
ip access-list extended IT_ACL
 ! 1. Unrestricted Access to Data Center
 permit ip 10.0.20.0 0.0.0.255 10.0.60.0 0.0.0.255      ! Server VLAN 60
 permit ip 10.0.20.0 0.0.0.255 10.0.70.0 0.0.0.255      ! Server VLAN 70

 ! 2. Infrastructure Management (SSH)
 permit tcp 10.0.20.0 0.0.0.255 10.0.0.0 0.0.0.255 eq 22 ! SSH to Management Network

 ! 3. Cross-Departmental Troubleshooting Access

 permit ip 10.0.20.0 0.0.0.255 10.0.10.0 0.0.0.255      ! Access to HR Dept
 permit ip 10.0.20.0 0.0.0.255 10.0.20.0 0.0.0.255      ! Local IT Subnet
 permit ip 10.0.20.0 0.0.0.255 10.0.30.0 0.0.0.255      ! Access to Sales Dept
 permit ip 10.0.20.0 0.0.0.255 10.0.40.0 0.0.0.255      ! Access to Support Depts

 ! 4. Permit all other traffic (e.g., Internet Access)
 permit ip any any

! Apply to Interface
interface Vlan20
 ip access-group IT_ACL in
```

#### 3. **Policy Logic (SALES_ACL):**

* **Permitted:** Explicit access is granted to necessary business services like the Corporate Web Server (HTTP/HTTPS), Email (SMTP/POP3), and time synchronization (NTP)

* **Denied:** Sales is prevented from reaching the HR Department subnet, as well as infrastructure servers (TFTP, Syslog, FTP)

* **Application:** Applied Inbound on the SALES VLAN (SVI 30) on the Distribution Switches in Office B.

```cisco
ip access-list extended SALES_ACL
 ! 1. Deny Traffic to HR Department
 10 deny ip 10.0.30.0 0.0.0.255 10.0.10.0 0.0.0.255     

 ! 2. Explicitly Permit Business Services (Data Center)
 20 permit tcp 10.0.30.0 0.0.0.255 host 10.0.60.4 eq 80    ! WWW Server (HTTP)
 30 permit tcp 10.0.30.0 0.0.0.255 host 10.0.60.4 eq 443   ! WWW Server (HTTPS)
 40 permit tcp 10.0.30.0 0.0.0.255 host 10.0.60.5 eq 25    ! Email Server (SMTP)
 50 permit tcp 10.0.30.0 0.0.0.255 host 10.0.60.5 eq 110   ! Email Server (POP3)
 60 permit udp 10.0.30.0 0.0.0.255 host 10.0.70.7 eq 123   ! NTP Server

 ! 3. Deny Infrastructure / Management Servers
 70 deny udp 10.0.30.0 0.0.0.255 host 10.0.70.5            ! TFTP Server
 80 deny udp 10.0.30.0 0.0.0.255 host 10.0.70.8            ! Syslog Server
 90 deny tcp 10.0.30.0 0.0.0.255 host 10.0.70.4            ! FTP Server

 ! 4. Permit all other traffic (e.g., Internet Access)
 100 permit ip any any

! Apply to Interface
interface Vlan30
 ip access-group SALES_ACL in
```

#### 4. **Policy Logic (SUPPORT_ACL):**

* **Permitted:** Explicit access is granted to necessary business services like the Corporate Web Server (HTTP/HTTPS), Email (SMTP/POP3), time synchronization (NTP) and other departamens (HR, SALES)

* **Denied:** Prevented from accessing core infrastructure and management servers (TFTP, Syslog, FTP).

* **Application:** Applied Inbound on the Support VLAN (SVI 40) on the Distribution Switches in Office B.

```cisco
ip access-list extended SUPPORT_ACL
 ! 1. Explicitly Permit Business Services (Data Center)
 10 permit tcp 10.0.40.0 0.0.0.255 host 10.0.60.4 eq 80    ! WWW Server (HTTP)
 20 permit tcp 10.0.40.0 0.0.0.255 host 10.0.60.4 eq 443   ! WWW Server (HTTPS)
 30 permit tcp 10.0.40.0 0.0.0.255 host 10.0.60.5 eq 25    ! Email Server (SMTP)
 40 permit tcp 10.0.40.0 0.0.0.255 host 10.0.60.5 eq 110   ! Email Server (POP3)
 50 permit udp 10.0.40.0 0.0.0.255 host 10.0.70.7 eq 123   ! NTP Server

 ! 2. Deny Infrastructure / Management Servers
 60 deny udp 10.0.40.0 0.0.0.255 host 10.0.70.5            ! TFTP Server
 70 deny udp 10.0.40.0 0.0.0.255 host 10.0.70.8            ! Syslog Server
 80 deny tcp 10.0.40.0 0.0.0.255 host 10.0.70.4            ! FTP Server

 ! 3. Permit all other traffic (Inter-departmental & Internet Access)
 100 permit ip any any

! Apply to Interface
interface Vlan40
 ip access-group SUPPORT_ACL in
```

#### 5. **Policy Logic (DC_SERVICES_OUT):**

* **Permitted:**
  - The IT Department (VLAN 20) is granted full, unrestricted IP access for management and maintenance.
    
  - Other users (or external traffic) are only permitted to reach specific servers on specific ports (HTTP/HTTPS for the Web Server, SMTP/POP3 for the Email Server).
    
  - The established keyword is utilized to allow return TCP traffic for sessions initiated from within the Data Center (e.g., server updates), without opening inbound ports unnecessarily.
  
* **Denied:** Drops unauthorized access attempts (such as Ping or SSH attempts from non-IT subnets).

* **Application:** Applied Outbound on the SRV_PUB VLAN (SVI 60) on the Distribution Switches in Data Center.

```cisco
ip access-list extended DC_SERVICES_OUT
 ! 1. Unrestricted Access for IT Administrators
 10 permit ip 10.0.20.0 0.0.0.255 any

 ! 2. Public / Internal Business Services
 20 permit tcp any host 10.0.60.4 eq 80    ! Web Server (HTTP)
 30 permit tcp any host 10.0.60.4 eq 443   ! Web Server (HTTPS)
 40 permit tcp any host 10.0.60.5 eq 25    ! Email Server (SMTP)
 50 permit tcp any host 10.0.60.5 eq 110   ! Email Server (POP3)

 ! 3. Allow Return Traffic for Established TCP Sessions
 60 permit tcp any any established

 ! 4. Explicitly Deny All Other Traffic
 100 deny ip any any

! Apply to Data Center Interface
interface Vlan60
 ip access-group DC_SERVICES_OUT out

```

#### 6. **Policy Logic (DC_INFRA_OUT):**

* **Permitted:**
  - The IT Department (VLAN 20) is granted full, unrestricted IP access for management and maintenance.
    
  - DHCP (UDP 67/68) and NTP (UDP 123) are permitted from any source,
    
  - Syslog traffic (UDP 514) is restricted exclusively to internal devices within the 10.0.0.0/8 corporate network
  
* **Denied:** other traffic like unauthorized FTP or TFTP access

* **Application:** Applied Outbound on the SRV_INT VLAN (SVI 70) on the Distribution Switches in Data Center.

```cisco

ip access-list extended DC_INFRA_OUT
 ! 1. Unrestricted Access for IT Administrators
 10 permit ip 10.0.20.0 0.0.0.255 any

 ! 2. Essential Network Services (DHCP & NTP)
 20 permit udp any host 10.0.70.6 eq 67    ! DHCP Server 
 21 permit udp any host 10.0.70.6 eq 68    ! DHCP Server
 30 permit udp any host 10.0.70.7 eq 123   ! NTP Server

 ! 3. Secured Logging (Syslog)
 40 permit udp 10.0.0.0 0.255.255.255 host 10.0.70.8 eq 514

 ! 4. Explicitly Deny All Other Traffic
 100 deny ip any any

! Apply to Data Center Interface
interface Vlan70
 ip access-group DC_INFRA_OUT out

```

#### 7. **Policy Logic (INTERNET_IN):**

* **Permitted:** Non-private IP addresses and access to DNS Server
  
* **Denied:** Drops incoming packets from the Internet that falsely claim to originate from private RFC 1918 IP addresses

* **Application:** Applied Inbound on the WAN/External interface (g0/0/0) of the Edge Router

``` cisco
  ip access-list extended INTERNET_IN
 ! 1. Anti-Spoofing (Block RFC 1918 from entering via Internet)
 deny ip 10.0.0.0 0.255.255.255 any
 deny ip 172.16.0.0 0.15.255.255 any
 deny ip 192.168.0.0 0.0.255.255 any

 ! 2. Explicitly Allow DNS
 permit udp host 8.8.8.8 any eq 53

 ! 3. Permit all other valid public traffic
 permit ip any any

! Apply to Edge Router WAN Interface
interface g0/0/0
 ip access-group INTERNET_IN in
```



### C. NAT Traffic Identification

A Standard ACL is used on the Edge Router to define which internal subnets are allowed to be translated via PAT (Port Address Translation) for Internet access.

```cisco
access-list 1 permit 10.0.10.0 0.0.0.255
access-list 1 permit 10.0.20.0 0.0.0.255
access-list 1 permit 10.0.100.0 0.0.0.255
access-list 1 permit 10.0.30.0 0.0.0.255
access-list 1 permit 10.0.40.0 0.0.0.255
access-list 1 permit 10.0.101.0 0.0.0.255
access-list 1 permit 10.0.60.0 0.0.0.255

ip nat inside source list 1 interface g0/0/0
```

## 3. Verification

#### Checking for active access lists and verifying hit counts to ensure traffic is being correctly permitted or dropped.

**Command:** `show access-lists`

<img width="936" height="550" alt="image" src="https://github.com/user-attachments/assets/23d2ca94-c824-4142-8b3b-946edb29a56d" />

#### Attempting to reach a Sales host (10.0.30.12) from an HR workstation

**Command:** `ping 10.0.30.12`

<img width="737" height="296" alt="image" src="https://github.com/user-attachments/assets/b8b7bb02-9631-4cd0-8971-41b46642c12b" />





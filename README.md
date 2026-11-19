# Enterprise-Multi-Department-Network
Multi-department enterprise network topology in Cisco Packet Tracer featuring EIGRP, OSPF redistribution, LACP EtherChannel, Inter-VLAN routing, and Extended ACLs.
# Multi-Department Enterprise Network Architecture & Zero-Trust Security

![Topology Status](https://shields.io)
![Routing](https://shields.io)
![Switching](https://shields.io)
![Security](https://shields.io)

## 📌 1. Project Overview
This repository contains the complete Cisco Packet Tracer simulation (`jockey2exp.pkt`) and architectural documentation for a multi-branch enterprise network. Designed to reflect real-world corporate infrastructure, the topology connects four distinct organizational departments (**IT**, **Outsourcing**, **Management**, and **Finance/HR**) to a centralized **Server Farm** using multi-protocol dynamic routing, Layer 2 high availability, and zero-trust access control policies.

---

## 🏗️ 2. Network Architecture & IP Addressing Plan

### Core & Departmental Routing Domains
* **IT Department (EIGRP Domain 1 — `172.16.0.0/21`):** Supports Developer, Network, Cyber, and IT Support teams across four `/23` subnets (`172.16.0.0/23`, `172.16.2.0/23`, `172.16.4.0/23`, `172.16.6.0/23`) interconnected via `/30` point-to-point transit links (`192.168.1.0/30` – `192.168.1.16/30`).
* **Outsourcing Department (EIGRP Domain 2 — `172.16.8.0/21`):** Supports Quality Management and Outsourcing teams across four `/23` subnets (`172.16.8.0/23`, `172.16.10.0/23`, `172.16.12.0/23`, `172.16.14.0/23`) over `/30` transit links (`192.168.1.20/30` – `192.168.1.32/30`).
* **Core ASBR (`Router5`):** Performs two-way dynamic route redistribution between the EIGRP domains and OSPF Area 1, providing seamless end-to-end reachability across different routing protocols.
* **Management Branch (OSPF Area 1 — `172.16.16.0/23`):** Connects executive workstations (`Router6`) directly into the OSPF backbone.
* **Finance & HR Branch (OSPF Area 1 — `172.16.18.0/24` & `172.16.19.0/24`):** Segmented at Layer 2 using VLANs and Router-on-a-Stick (`Router7`).

### Layer 2 VLAN Segmentation & Server Farm Table

| Segment / Service | VLAN ID | Subnet / Mask | Default Gateway | Security & Access Control Policy |
| :--- | :--- | :--- | :--- | :--- |
| **Finance Department** | VLAN 10 | `172.16.18.0/24` | `172.16.18.1` | HTTPS (`443`) to Finance Server (`192.168.1.134`) only |
| **HR Department** | VLAN 20 | `172.16.19.0/24` | `172.16.19.1` | HTTPS (`443`) to HR Server (`192.168.1.136`) only; blocked from VLAN 10 |
| **Trunk Native VLAN** | VLAN 99 | Unused | N/A | Dedicated native VLAN to mitigate VLAN hopping attacks |
| **Centralized Server Farm** | Native | `192.168.1.128/26` | `192.168.1.129` | Protected by outbound Extended ACL `svr-1` on `Router7 Fa1/0` |

### Centralized Server Farm Allocation (`192.168.1.128/26`)
* **Management Server (`192.168.1.130`):** Accessible strictly by Management (`172.16.16.0/23`) over HTTPS (`TCP/443`).
* **DNS Server (`192.168.1.131`):** Accessible by all internal departments over DNS (`UDP/53`).
* **IT Department Server (`192.168.1.133`):** Accessible strictly by IT subnets (`172.16.0.0/21`) over HTTPS (`TCP/443`).
* **Finance Server (`192.168.1.134`):** Accessible strictly by Finance VLAN 10 (`172.16.18.0/24`) over HTTPS (`TCP/443`).
* **Outsourcing Server (`192.168.1.135`):** Accessible strictly by Outsourcing subnets (`172.16.8.0/21`) over HTTPS (`TCP/443`).
* **HR Server (`192.168.1.136`):** Accessible strictly by HR VLAN 20 (`172.16.19.0/24`) over HTTPS (`TCP/443`).

---

## ⚙️ 3. Key Technologies & Security Controls
* **Multi-Protocol Dynamic Routing:** Configured EIGRP across regional branches and OSPF Area 1 across corporate and server edge routers, using `Router5` as an Autonomous System Boundary Router (ASBR).
* **High-Availability Switching (LACP & Rapid-PVST+):** Bundled dual FastEthernet links (`Fa0/23`–`Fa0/24`) between `Switch-Fin` and `Switch-HR` into an active **LACP EtherChannel (`Port-channel 1`)** running **Rapid-PVST+**, doubling inter-switch bandwidth and eliminating single points of failure.
* **Access Layer Port Hardening:** Enabled **PortFast**, **BPDU Guard**, and **Sticky Port Security** (`maximum 2`, `violation restrict`) on all employee access ports to mitigate rogue switches and MAC flooding.
* **Zero-Trust Access Control Lists (ACLs):**
  * **Server Edge Policy (`svr-1`):** Applied **outbound** on `Router7 Fa1/0` to inspect all incoming departmental requests before reaching the servers while allowing local DHCP broadcasts and return traffic to pass unimpeded.
  * **Inter-VLAN Isolation (`BLOCK-HR-TO-FINANCE`):** Applied **inbound** on sub-interface `Fa0/1.20` to prevent lateral workstation traffic from HR (`172.16.19.0/24`) to Finance (`172.16.18.0/24`).

---

## 💻 4. Core Configuration Highlights

### Router7 — Router-on-a-Stick, DHCP & Zero-Trust ACLs
```cisco
interface FastEthernet0/1.10
 description Finance_VLAN_10
 encapsulation dot1Q 10
 ip address 172.16.18.1 255.255.255.0

interface FastEthernet0/1.20
 description HR_VLAN_20
 encapsulation dot1Q 20
 ip address 172.16.19.1 255.255.255.0
 ip access-group BLOCK-HR-TO-FINANCE in

ip access-list extended BLOCK-HR-TO-FINANCE
 deny ip 172.16.19.0 0.0.0.255 172.16.18.0 0.0.0.255
 permit ip any any

ip access-list extended svr-1
 10 permit ospf any any
 20 permit udp any host 192.168.1.131 eq domain
 30 permit tcp 172.16.2.0 0.0.1.255 host 192.168.1.133 eq 443
 40 permit tcp 172.16.6.0 0.0.1.255 host 192.168.1.133 eq 443
 50 permit tcp 172.16.4.0 0.0.1.255 host 192.168.1.133 eq 443
 60 permit tcp 172.16.0.0 0.0.1.255 host 192.168.1.133 eq 443
 70 permit tcp 172.16.8.0 0.0.1.255 host 192.168.1.135 eq 443
 80 permit tcp 172.16.14.0 0.0.1.255 host 192.168.1.135 eq 443
 90 permit tcp 172.16.12.0 0.0.1.255 host 192.168.1.135 eq 443
 100 permit tcp 172.16.10.0 0.0.1.255 host 192.168.1.135 eq 443
 110 permit tcp 172.16.16.0 0.0.1.255 host 192.168.1.130 eq 443
 120 permit tcp 172.16.18.0 0.0.0.255 host 192.168.1.134 eq 443
 125 permit tcp 172.16.19.0 0.0.0.255 host 192.168.1.136 eq 443
 130 deny ip any any

interface FastEthernet1/0
 ip access-group svr-1 out
```

### Switch-Fin & Switch-HR — LACP EtherChannel & Access Hardening
```cisco
spanning-tree mode rapid-pvst

interface range FastEthernet0/23 - 24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 channel-group 1 mode active

interface Port-channel 1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99

interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
```

---

## 🛠️ 5. Engineering Challenges & Troubleshooting Log
1. **ACL Direction & DHCP Broadcast Drops:** Applying `svr-1` inbound on `Router7 Fa0/1` initially dropped client DHCP Discover broadcasts (`0.0.0.0` to `255.255.255.255`), while inbound placement on `Fa1/0` blocked server replies. **Fix:** Moved `svr-1` **outbound** on `Fa1/0` (Server Farm interface), enabling local DHCP and server responses while enforcing a single centralized security policy for all incoming server requests.
2. **LACP Negotiation Mismatch (`Po1(SD)`):** Configuring `channel-group 1 mode active` on `Switch-HR` before `Switch-Fin` caused member ports to enter Stand-alone `(I)` state. **Fix:** Synchronized LACP `active` mode and Native VLAN `99` trunk parameters across both switches to establish `Po1(SU)`.
3. **Simulator DHCP Snooping Constraint on Port-Channels:** Cisco Packet Tracer 2960 switches do not support `ip dhcp snooping trust` directly on logical `Port-channel` interfaces, dropping valid DHCP offers traversing the LACP bundle. **Fix:** Documented the simulator limitation and prioritized LACP EtherChannel redundancy and Sticky Port Security across the multi-switch trunk.

---

## 🚀 6. How to Run This Lab
1. Download **`jockey2exp.pkt`** from this repository.
2. Open the file using **Cisco Packet Tracer (v8.0 or higher)**.
3. Verify DHCP leases on Finance (`172.16.18.x`) and HR (`172.16.19.x`) endpoints, test HTTPS (`443`) access to departmental servers via the desktop web browser, and inspect live ACL hit counts via `show ip access-lists svr-1` on `Router7`.

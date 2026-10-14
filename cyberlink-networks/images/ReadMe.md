# CyberLink Networks
### Securing Your Digital Infrastructure

**Enterprise Network Infrastructure Project**

Design and implementation of a secure, multi-branch enterprise network featuring dynamic routing, centralized services, internet connectivity simulation, and layered security controls, built in Cisco Packet Tracer.

| | |
|---|---|
| **Prepared by** | Ahmad Madi |
| **Institution** | University College of Applied Sciences (UCAS) |
| **Program** | Cyber Security Engineering |
| **Course** | Computer Networking (COMP3115) |
| **Date** | October 2026 |

## Table of Contents
1. [Introduction](#1-introduction)
2. [Network Topology Overview](#2-network-topology-overview)
3. [Branch Overview](#3-branch-overview)
4. [IP Addressing Scheme](#4-ip-addressing-scheme)
5. [Dynamic Routing (OSPF)](#5-dynamic-routing-ospf)
6. [Centralized DHCP Services](#6-centralized-dhcp-services)
7. [DNS Services](#7-dns-services)
8. [Internal Web Hosting](#8-internal-web-hosting)
9. [Internet Access Simulation (NAT / PAT)](#9-internet-access-simulation-nat--pat)
10. [Security Implementation](#10-security-implementation)
11. [Testing & Verification Summary](#11-testing--verification-summary)
12. [Conclusion](#12-conclusion)

---

## 1. Introduction

This project presents the design and full implementation of a multi-branch enterprise network for **CyberLink Networks**, simulated entirely in Cisco Packet Tracer. The network connects five independent branches (Human Resources, Sales, Development, a central Data Center, and a Remote Office) through a central core router using dynamic routing. The project demonstrates practical, hands-on mastery of enterprise networking concepts including IP addressing (VLSM), dynamic routing, centralized network services, internet connectivity simulation, and layered network security.

### Project Objectives
- **Connectivity:** Establish full routed connectivity between five branches through a central hub router.
- **Automation:** Deploy centralized DHCP for automatic IP assignment across all branches.
- **Name Resolution:** Deploy a DNS server to resolve internal and external domain names.
- **Hosting:** Host an internal corporate website accessible to all branches.
- **Internet Simulation:** Simulate real-world internet access using NAT/PAT through a simulated ISP.
- **Security:** Apply Access Control Lists, Port Security, and SSH-based device hardening.

![Figure 1: Full project logical topology](images/images/figure-01.png)
*Figure 1: Full project logical topology*

## 2. Network Topology Overview

The network follows a **Hub-and-Spoke** topology. A central router, R-CORE, connects to each branch router over a dedicated point-to-point WAN link. R-CORE also connects outward to R-ISP, which simulates the public internet, through a link representing the service provider connection.

### Core Infrastructure Components
| Device | Role |
|--------|------|
| R-CORE | Central router: OSPF hub, NAT gateway |
| R-HR / R-Sales / R-Dev | Branch routers for HR, Sales, and Development |
| R-Main | Main Data Center router: hosts centralized services |
| R-Remote | Remote Office router: wired & wireless access |
| R-ISP | Simulated Internet Service Provider edge router |

## 3. Branch Overview

| Branch | Network | Devices | Notes |
|--------|---------|---------|-------|
| Human Resources (HR) | 192.168.10.0/29 | PC, Laptop, Server, Printer | Restricted from accessing Development |
| Sales | 192.168.11.0/29 | PC, Laptop, Server, Printer | - |
| Development | 192.168.12.0/29 | PC, Laptop, Server, Printer | Protected by ACL from HR |
| Main Data Center | 192.168.13.0/28 | 3 Servers, PCs, Laptop, Printer | Hosts DHCP, DNS, Web services |
| Remote Office | 192.168.14.0/24 | PC, Laptop, Tablet, Smartphone | Wi-Fi secured with WPA2-PSK |

## 4. IP Addressing Scheme

The network uses VLSM subnetting to efficiently allocate address space per branch, with /30 point-to-point links for all WAN connections.

### LAN Networks
| Branch | Network | Subnet Mask | Gateway |
|--------|---------|-------------|---------|
| HR | 192.168.10.0 | 255.255.255.248 | 192.168.10.1 |
| Sales | 192.168.11.0 | 255.255.255.248 | 192.168.11.1 |
| Development | 192.168.12.0 | 255.255.255.248 | 192.168.12.1 |
| Main | 192.168.13.0 | 255.255.255.240 | 192.168.13.1 |
| Remote Office | 192.168.14.0 | varies (/30, /29) | 192.168.14.1 / .5 / .9 |

### WAN Links (Point-to-Point /30)
| Link | Network |
|------|---------|
| R-HR ↔ R-CORE | 10.0.0.0/30 |
| R-Sales ↔ R-CORE | 10.0.0.4/30 |
| R-Dev ↔ R-CORE | 10.0.0.8/30 |
| R-Main ↔ R-CORE | 10.0.0.12/30 |
| R-Remote ↔ R-CORE | 10.0.0.16/30 |
| R-CORE ↔ R-ISP | 200.100.100.0/30 |

## 5. Dynamic Routing (OSPF)

All routers run **OSPF Area 0** (single-area), allowing every router to learn all branch networks dynamically. Each router advertises its directly connected networks, and R-CORE originates a default route toward R-ISP for internet-bound traffic, which the branch routers learn through OSPF.

```
router ospf 1
 network ... area 0
 default-information originate
```

![Figure 2: show ip route output on R-CORE](images/images/figure-02.png)
*Figure 2: "show ip route" output on R-CORE*

![Figure 3: show ip route on R-Sales](images/images/figure-03.png)
*Figure 3: "show ip route" on R-Sales: all branch networks and the default route learned via OSPF*

## 6. Centralized DHCP Services

A DHCP server is deployed on **Server5** within the Main Data Center, providing automatic IP assignment for all five branches. Since the server resides remotely from each branch's LAN, the `ip helper-address` command relays DHCP broadcasts across the routed network to the central server.

### DHCP Pools Configured
| Pool | Gateway | DNS Server | Range Start |
|------|---------|------------|-------------|
| HR-Pool | 192.168.10.1 | 192.168.13.2 | 192.168.10.2 |
| Sales-Pool | 192.168.11.1 | 192.168.13.2 | 192.168.11.2 |
| Dev-Pool | 192.168.12.1 | 192.168.13.2 | 192.168.12.2 |
| Main-Pool | 192.168.13.1 | 192.168.13.2 | 192.168.13.2 |
| Remote-Pool | 192.168.14.1 | 192.168.13.2 | 192.168.14.2 |

![Figure 4: DHCP pool configuration on Server5](images/images/figure-04.png)
*Figure 4: DHCP pool configuration on Server5*

![Figure 5: HR client receiving an address via DHCP](images/images/figure-05.png)
*Figure 5: HR client set to DHCP and receiving 192.168.10.4 with the correct gateway and DNS server*

## 7. DNS Services

A DNS server on **Server3** (192.168.13.2) resolves friendly domain names to IP addresses for both internal and external resources, allowing users to browse by name instead of memorizing raw IP addresses.

| Domain Name | Resolves To | Purpose |
|-------------|-------------|---------|
| www.netguard.com | 192.168.13.2 | Internal corporate website |
| www.globalnet.com | 200.100.200.2 | Simulated external internet site |

![Figure 6: DNS records configuration on Server3](images/images/figure-06.png)
*Figure 6: DNS records configuration on Server3*

![Figure 7: nslookup resolving www.netguard.com](images/images/figure-07.png)
*Figure 7: nslookup resolving www.netguard.com*

## 8. Internal Web Hosting

**Server3** in the Main Data Center also hosts the company's internal website, styled with the CyberLink Networks brand identity and accessible from any branch via the internal DNS record.

![Figure 8: Internal corporate website](images/images/figure-08.png)
*Figure 8: Internal corporate website opened through www.netguard.com*

## 9. Internet Access Simulation (NAT / PAT)

To simulate real internet connectivity, R-CORE performs **NAT Overload (PAT)**, translating all private internal addresses to its single public-facing IP (**200.100.100.1**) when communicating with the simulated ISP network through R-ISP. An external server behind R-ISP represents a public internet resource.

```
ip nat inside
ip nat outside
ip nat inside source list 1 interface Gi0/0 overload
```

![Figure 9: show ip nat translations](images/images/figure-09.png)
*Figure 9: "show ip nat translations" with active entries*

![Figure 10: Client browsing the external simulated website](images/images/figure-10.png)
*Figure 10: Client browsing the external simulated website*

## 10. Security Implementation

### 10.1 Access Control Lists (ACL)
An extended ACL on R-Dev blocks traffic originating from the HR subnet toward the Development subnet, enforcing department-level isolation while allowing all other traffic.

![Figure 11: show access-lists on R-Dev](images/images/figure-11.png)
*Figure 11: "show access-lists" on R-Dev*

### 10.2 Port Security
Every access-layer switch port is configured with Port Security, allowing only one learned MAC address per port and shutting the port down automatically if a violation occurs.

![Figure 12: show port-security output](images/images/figure-12.png)
*Figure 12: "show port-security" output*

### 10.3 Device Hardening (SSH)
Routers are hardened with an encrypted enable secret and SSH-based remote management, replacing insecure Telnet access.

![Figure 13: show ip ssh](images/images/figure-13.png)
*Figure 13: "show ip ssh" confirming SSH enabled*

### 10.4 Wireless Security
The Remote Office Access Point uses WPA2-PSK with AES encryption, replacing the default open configuration.

![Figure 14: Access Point wireless security](images/images/figure-14.png)
*Figure 14: Access Point configured with WPA2-PSK and AES (passphrase hidden)*

## 11. Testing & Verification Summary

| ✓ | Test | Expected Result |
|---|------|-----------------|
| ✓ | DHCP client request | Automatic IP assignment from correct pool (Figure 5) |
| ✓ | DNS resolution | www.netguard.com resolves to 192.168.13.2 (Figure 7) |
| ✓ | Internal website access | Website loads via domain name (Figure 8) |
| ✓ | External website access (NAT) | Translation entries appear in NAT table (Figures 9, 10) |
| ✓ | HR → Development ping | Blocked (Destination host unreachable) (Figure 15) |
| ✓ | SSH enabled on routers | SSH enabled on R-CORE (Figure 13) |
| ✓ | Port security configured | Max 1 MAC per port, shutdown action (Figure 12) |

![Figure 15: HR to Development ping blocked by ACL](images/images/figure-15.png)
*Figure 15: HR → Development ping blocked by ACL*

## 12. Conclusion

This project successfully delivers a fully functional, secure, multi-branch enterprise network. Through the integration of VLSM addressing, OSPF dynamic routing, centralized DHCP and DNS services, internal web hosting, simulated internet access via NAT/PAT, and a layered security model combining ACLs, Port Security, and SSH hardening, the network reflects practical, industry-aligned enterprise design principles.

The project demonstrates not only core networking competency but also a security-first mindset consistent with the Cyber Security Engineering discipline, making it a strong practical foundation for further study in network defense and infrastructure protection.

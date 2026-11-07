# Enterprise Bank Network Design and Implementation

A secure, scalable, and redundant enterprise network for a four-floor banking building, designed and simulated in **Cisco Packet Tracer**. The design uses a hierarchical **Core / Distribution / Access** model, **OSPF** dynamic routing, **VLANs** with **VLSM** subnetting, centralized **DHCP**, and layered security hardening.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Features](#features)
3. [Network Architecture](#network-architecture)
4. [Department and VLAN Plan](#department-and-vlan-plan)
5. [IP Addressing Plan](#ip-addressing-plan)
6. [Configuration Summary](#configuration-summary)
7. [Testing and Verification](#testing-and-verification)
8. [Repository Structure](#repository-structure)
9. [How to Run](#how-to-run)
10. [Tools Used](#tools-used)
11. [Skills Demonstrated](#skills-demonstrated)
12. [Author](#author)

---

## Project Overview

A banking and insurance company is opening a new branch in a four-story building and needs a complete network design. Each floor hosts three departments (the fourth floor also hosts the Server Room). The network must support wired and wireless users, automatic IP assignment, inter-department communication, and secure remote management, with redundancy built in.

**Key requirements**

- One router per floor, connecting that floor's switches
- OSPF as the routing protocol
- Hierarchical design (core, distribution, access) with redundancy
- A separate VLAN and subnet for every department
- Around 60 users (wired and wireless) per department, except the Server Room
- Dynamic IPv4 addressing from dedicated DHCP servers in the Server Room
- HTTP and Email servers
- SSH on all routers
- Basic device hardening and port security

---

## Features

- Three-layer hierarchical topology with redundant links
- 4 routers (one per floor) running OSPF
- 4 Layer 3 switches (one per floor) for distribution and inter-VLAN routing
- 12 VLANs, one per department, each on its own subnet
- VLSM subnetting from the `192.168.10.0` base address
- /30 point-to-point links between routers and Layer 3 switches
- Centralized DHCP, HTTP, and Email servers in the Server Room
- Wireless access point in every department
- SSH remote management on all routers
- Port security with sticky MAC addresses and shutdown violation mode
- Basic hardening: hostnames, console and VTY passwords, login banners, disabled DNS lookup

---

## Network Architecture

| Layer | Devices | Role |
|---|---|---|
| **Core / Routing** | Floor1 to Floor4 routers (Cisco 2911) | OSPF routing between floors, redundant inter-router links |
| **Distribution** | Floor1 to Floor4 switches (Cisco 3650-24PS) | Layer 3 aggregation, inter-VLAN routing, uplinks to routers |
| **Access** | Department switches (Cisco 2960-24TT) and access points | End-device connectivity (PCs, printers, laptops, servers) |

**Redundancy:** routers are linked to each other, and each distribution switch connects to the access switches and routers over multiple paths, so a single link or device failure does not isolate a floor.

### Floor Layout

| Floor | Departments |
|---|---|
| First | Management, Research, Human Resource |
| Second | Marketing, Accounting, Finance |
| Third | Logistics and Store, Customer Care, Guest Area |
| Fourth | Administration, ICT, Server Room |

---

## Department and VLAN Plan

| Floor | Department | VLAN ID | PCs | Printers |
|---|---|---|---|---|
| 1 | Management | 10 | 20 | 4 |
| 1 | Research | 20 | 20 | 4 |
| 1 | Human Resource | 30 | 20 | 4 |
| 2 | Marketing | 40 | 20 | 4 |
| 2 | Accounting | 50 | 20 | 4 |
| 2 | Finance | 60 | 20 | 4 |
| 3 | Logistics and Store | 70 | 20 | 4 |
| 3 | Customer Care | 80 | 20 | 4 |
| 3 | Guest Area | 90 | 40 | 2 |
| 4 | Administration | 100 | 20 | 2 |
| 4 | ICT | 110 | 20 | 2 |
| 4 | Server Room | 120 | 2 Admin PCs | 3 servers (DHCP, HTTP, Email) |

---

## IP Addressing Plan

**Base network:** `192.168.10.0`
**Subnet mask for all VLANs:** `255.255.255.192` (`/26`, 62 usable hosts per subnet)

A /26 comfortably covers the ~60 users required per department (wired and wireless).

| VLAN | Department | Network Address | Usable Range | Broadcast |
|---|---|---|---|---|
| 10 | Management | 192.168.10.0/26 | 192.168.10.1 - 192.168.10.62 | 192.168.10.63 |
| 20 | Research | 192.168.10.64/26 | 192.168.10.65 - 192.168.10.126 | 192.168.10.127 |
| 30 | Human Resource | 192.168.10.128/26 | 192.168.10.129 - 192.168.10.190 | 192.168.10.191 |
| 40 | Marketing | 192.168.10.192/26 | 192.168.10.193 - 192.168.10.254 | 192.168.10.255 |
| 50 | Accounting | 192.168.11.0/26 | 192.168.11.1 - 192.168.11.62 | 192.168.11.63 |
| 60 | Finance | 192.168.11.64/26 | 192.168.11.65 - 192.168.11.126 | 192.168.11.127 |
| 70 | Logistics and Store | 192.168.11.128/26 | 192.168.11.129 - 192.168.11.190 | 192.168.11.191 |
| 80 | Customer Care | 192.168.11.192/26 | 192.168.11.193 - 192.168.11.254 | 192.168.11.255 |
| 90 | Guest Area | 192.168.12.0/26 | 192.168.12.1 - 192.168.12.62 | 192.168.12.63 |
| 100 | Administration | 192.168.12.64/26 | 192.168.12.65 - 192.168.12.126 | 192.168.12.127 |
| 110 | ICT | 192.168.12.128/26 | 192.168.12.129 - 192.168.12.190 | 192.168.12.191 |
| 120 | Server Room | 192.168.12.192/26 | 192.168.12.193 - 192.168.12.254 | 192.168.12.255 |

## Configuration Summary

### Basic Device Configuration

- Unique hostnames on all devices
- Console and VTY line passwords
- Login banner messages
- `no ip domain-lookup` to disable DNS lookup on mistyped commands

### VLANs and Trunking

- A VLAN per department, created on every relevant switch
- Trunk links between access switches and distribution switches
- Access ports assigned to the department VLAN

### Routing

- **OSPF** configured on all floor routers and Layer 3 switches
- All VLAN subnets and /30 links advertised, giving full reachability between departments

### DHCP

- DHCP server in the Server Room (VLAN 120) with a pool per VLAN
- Relay (`ip helper-address`) on the VLAN gateways so hosts in every VLAN receive addresses automatically

### Services

- **DHCP server:** dynamic addressing for all hosts
- **HTTP server:** internal web service
- **Email server:** internal mail service

### Security

- **SSH** enabled on all routers for secure remote login
- **Port security** on access ports:
  - `switchport port-security`
  - `switchport port-security mac-address sticky`
  - `switchport port-security violation shutdown`

### Wireless

- An access point in every department so users can connect wirelessly alongside wired hosts

---

## Testing and Verification

| Test | Expected Result |
|---|---|
| Ping between hosts in the same VLAN | Success |
| Ping between hosts in different VLANs | Success (inter-VLAN routing and OSPF) |
| Ping across floors | Success |
| Hosts receive IP addresses via DHCP | Success |
| Browse the HTTP server from a host | Success |
| Send and receive email via the Email server | Success |
| SSH to a router | Success |
| Port security violation (unknown MAC on a secured port) | Port enters err-disabled state |

### Useful Verification Commands

```
show ip route
show ip ospf neighbor
show vlan brief
show interfaces trunk
show port-security interface <interface>
show ip dhcp binding
show running-config
```

---


## Tools Used

- **Cisco Packet Tracer**: design, configuration, and simulation
- **Cisco IOS CLI**: device configuration

---

## Skills Demonstrated

OSPF routing, VLANs and inter-VLAN routing, VLSM subnetting, DHCP and DHCP relay, SSH, port security, hierarchical network design, network redundancy, wireless networking, and technical documentation.

---

## License

This project is for educational purposes.

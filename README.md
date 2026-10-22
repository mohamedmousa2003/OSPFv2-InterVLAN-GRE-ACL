<h1 align="center">OSPFv2 with Inter-VLAN Routing, GRE Tunnel, and Standard ACL</h1>

<p align="center">
  A Cisco Packet Tracer lab that combines OSPFv2 routing across a service provider, Router-on-a-Stick inter-VLAN routing, a GRE tunnel between two sites, and a standard ACL.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white" alt="Cisco Packet Tracer"/>
  <img src="https://img.shields.io/badge/Routing-OSPFv2-red?style=for-the-badge" alt="OSPFv2"/>
  <img src="https://img.shields.io/badge/Tunnel-GRE-orange?style=for-the-badge" alt="GRE Tunnel"/>
  <img src="https://img.shields.io/badge/Security-Standard%20ACL-blue?style=for-the-badge" alt="Standard ACL"/>
  <img src="https://img.shields.io/badge/Level-CCNA-green?style=for-the-badge" alt="CCNA"/>
</p>

---

## Table of Contents

- [Overview](#overview)
- [Network Topology](#network-topology)
- [VLAN and Port Assignment](#vlan-and-port-assignment)
- [IP Addressing Table](#ip-addressing-table)
- [Configuration](#configuration)
- [Verification and Testing](#verification-and-testing)
- [Screenshots](#screenshots)
- [Repository Structure](#repository-structure)
- [How to Run](#how-to-run)
- [Skills Demonstrated](#skills-demonstrated)
- [Author](#author)

---

## Overview

This project connects a **Headquarters (HQ)** site and a **Branch** site through a **Service Provider (SP)** router. HQ has two VLANs (HR and Development) routed with **Router-on-a-Stick**. **OSPFv2** in Area 0 provides dynamic routing across all three routers, a **GRE tunnel** creates a logical point-to-point link between HQ and Branch, and a **standard ACL** on the Branch router blocks selected hosts.

This project builds on the [Inter-VLAN Routing](https://github.com/mohamedmousa2003/Inter-VLAN-Routing-Router-on-a-Stick) lab.

**Objectives**

- Segment the HQ network into VLAN 10 (HR) and VLAN 20 (Development).
- Configure inter-VLAN routing using sub-interfaces on a trunk link.
- Enable OSPFv2 on HQ, SP, and BRANCH.
- Build a GRE tunnel between HQ and BRANCH.
- Restrict specific Branch hosts using a standard ACL.

**Note on GRE:** GRE encapsulates traffic between the two sites but does **not** encrypt it. Adding IPsec on top of GRE would turn it into a secure VPN.

---

## Network Topology

| Component | Quantity | Model           |
|-----------|----------|-----------------|
| Routers   | 3        | Cisco 2911      |
| Switches  | 2        | Cisco 2960-24TT |
| Laptops   | 7        | Laptop-PT       |

```
   VLAN 10 (HR)                                                   Branch LAN
 [Laptop0]--+                                                   192.168.2.0/24
 [Laptop1]--+--[Switch0]--Trunk--[HQ]=====[SP]=====[BRANCH]--[Switch1]--+--[Laptop3]
            |                     |                    |               +--[Laptop4]
 [Laptop2]--+                     +--- GRE Tunnel0 ----+               +--[Laptop5]
 [Laptop6]--+
   VLAN 20 (Development)

   HQ <-- 2.2.2.0/30 --> SP <-- 3.3.3.0/30 --> BRANCH
   Tunnel0: 10.1.1.0/30 (HQ 10.1.1.1, BRANCH 10.1.1.2)
```

---

## VLAN and Port Assignment

| VLAN ID | VLAN Name   | Ports          | Mode   | Connected Devices |
|---------|-------------|----------------|--------|-------------------|
| 10      | HR          | Fa0/1 - Fa0/12  | Access | Laptop0, Laptop1  |
| 20      | Development | Fa0/13 - Fa0/24 | Access | Laptop2, Laptop6  |
| All     | N/A         | Gig0/1         | Trunk  | HQ (Gig0/0)       |

---

## IP Addressing Table

| Device  | Interface | IP Address    | Subnet Mask     | Default Gateway |
|---------|-----------|---------------|-----------------|-----------------|
| HQ      | Gig0/0.10 | 192.168.10.1  | 255.255.255.0   | N/A             |
| HQ      | Gig0/0.20 | 192.168.20.1  | 255.255.255.0   | N/A             |
| HQ      | Gig0/1    | 2.2.2.1       | 255.255.255.252 | N/A             |
| HQ      | Tunnel0   | 10.1.1.1      | 255.255.255.252 | N/A             |
| SP      | Gig0/0    | 2.2.2.2       | 255.255.255.252 | N/A             |
| SP      | Gig0/1    | 3.3.3.2       | 255.255.255.252 | N/A             |
| BRANCH  | Gig0/1    | 3.3.3.1       | 255.255.255.252 | N/A             |
| BRANCH  | Gig0/0    | 192.168.2.1   | 255.255.255.0   | N/A             |
| BRANCH  | Tunnel0   | 10.1.1.2      | 255.255.255.252 | N/A             |
| Laptop0 | Fa0       | 192.168.10.2  | 255.255.255.0   | 192.168.10.1    |
| Laptop1 | Fa0       | 192.168.10.3  | 255.255.255.0   | 192.168.10.1    |
| Laptop2 | Fa0       | 192.168.20.2  | 255.255.255.0   | 192.168.20.1    |
| Laptop6 | Fa0       | 192.168.20.3  | 255.255.255.0   | 192.168.20.1    |
| Laptop3 | Fa0       | 192.168.2.2   | 255.255.255.0   | 192.168.2.1     |
| Laptop4 | Fa0       | 192.168.2.3   | 255.255.255.0   | 192.168.2.1     |
| Laptop5 | Fa0       | 192.168.2.4   | 255.255.255.0   | 192.168.2.1     |

---

## Configuration

### Switch0: VLANs and Trunk

```
Switch(config)# vlan 10
Switch(config-vlan)# name HR
Switch(config-vlan)# exit
Switch(config)# vlan 20
Switch(config-vlan)# name Development
Switch(config-vlan)# exit

Switch(config)# interface range fastEthernet 0/1 - 12
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10
Switch(config-if-range)# exit

Switch(config)# interface range fastEthernet 0/13 - 24
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 20
Switch(config-if-range)# exit

Switch(config)# interface gigabitEthernet 0/1
Switch(config-if)# switchport mode trunk
```

### HQ: Sub-interfaces and WAN Link

```
HQ(config)# interface gigabitEthernet 0/0
HQ(config-if)# no shutdown
HQ(config-if)# exit
HQ(config)# interface gigabitEthernet 0/0.10
HQ(config-subif)# encapsulation dot1Q 10
HQ(config-subif)# ip address 192.168.10.1 255.255.255.0
HQ(config-subif)# exit
HQ(config)# interface gigabitEthernet 0/0.20
HQ(config-subif)# encapsulation dot1Q 20
HQ(config-subif)# ip address 192.168.20.1 255.255.255.0
HQ(config-subif)# exit
HQ(config)# interface gigabitEthernet 0/1
HQ(config-if)# ip address 2.2.2.1 255.255.255.252
HQ(config-if)# no shutdown
```

### SP and BRANCH: Interfaces

```
SP(config)# interface gigabitEthernet 0/0
SP(config-if)# ip address 2.2.2.2 255.255.255.252
SP(config-if)# no shutdown
SP(config-if)# exit
SP(config)# interface gigabitEthernet 0/1
SP(config-if)# ip address 3.3.3.2 255.255.255.252
SP(config-if)# no shutdown

BRANCH(config)# interface gigabitEthernet 0/0
BRANCH(config-if)# ip address 192.168.2.1 255.255.255.0
BRANCH(config-if)# no shutdown
BRANCH(config-if)# exit
BRANCH(config)# interface gigabitEthernet 0/1
BRANCH(config-if)# ip address 3.3.3.1 255.255.255.252
BRANCH(config-if)# no shutdown
```

### OSPFv2 Configuration

**HQ**
```
HQ(config)# router ospf 1
HQ(config-router)# passive-interface GigabitEthernet0/0.10
HQ(config-router)# passive-interface GigabitEthernet0/0.20
HQ(config-router)# network 192.168.10.0 0.0.0.255 area 0
HQ(config-router)# network 192.168.20.0 0.0.0.255 area 0
HQ(config-router)# network 2.2.2.0 0.0.0.3 area 0
```

**SP**
```
SP(config)# router ospf 1
SP(config-router)# network 2.2.2.0 0.0.0.3 area 0
SP(config-router)# network 3.3.3.0 0.0.0.3 area 0
```

**BRANCH**
```
BRANCH(config)# router ospf 1
BRANCH(config-router)# passive-interface GigabitEthernet0/0
BRANCH(config-router)# network 192.168.2.0 0.0.0.255 area 0
BRANCH(config-router)# network 3.3.3.0 0.0.0.3 area 0
```

> OSPF `network` statements use **wildcard masks**. For example, `255.255.255.252` becomes `0.0.0.3`. Each VLAN sub-interface needs its own `passive-interface` line because the physical interface setting does not cover sub-interfaces.

### GRE Tunnel

**HQ**
```
HQ(config)# interface tunnel 0
HQ(config-if)# ip address 10.1.1.1 255.255.255.252
HQ(config-if)# tunnel source gigabitEthernet 0/1
HQ(config-if)# tunnel destination 3.3.3.1
HQ(config-if)# tunnel mode gre ip
```

**BRANCH**
```
BRANCH(config)# interface tunnel 0
BRANCH(config-if)# ip address 10.1.1.2 255.255.255.252
BRANCH(config-if)# tunnel source gigabitEthernet 0/1
BRANCH(config-if)# tunnel destination 2.2.2.1
BRANCH(config-if)# tunnel mode gre ip
```

### Standard ACL on BRANCH

Blocks Laptop3 (192.168.2.2) and Laptop4 (192.168.2.3) and permits all other hosts. The wildcard mask `0.0.0.1` matches both addresses.

```
BRANCH(config)# access-list 99 deny 192.168.2.2 0.0.0.1
BRANCH(config)# access-list 99 permit any
BRANCH(config)# interface gigabitEthernet 0/0
BRANCH(config-if)# ip access-group 99 in
```

### Steps Performed

1. Created VLAN 10 and VLAN 20 and assigned access ports on Switch0.
2. Configured a trunk to HQ and 802.1Q sub-interfaces for inter-VLAN routing.
3. Assigned IP addresses to all router interfaces and laptops.
4. Enabled OSPFv2 in Area 0 on HQ, SP, and BRANCH.
5. Built a GRE tunnel between HQ and BRANCH.
6. Created and applied a standard ACL on BRANCH.
7. Verified routing, the tunnel, and the ACL with `show` commands and `ping`.

---

## Verification and Testing

| Command | Purpose |
|---------|---------|
| `show vlan brief` | Verify VLANs and assigned ports on Switch0 |
| `show interfaces trunk` | Confirm Gig0/1 is trunking |
| `show ip ospf neighbor` | Verify OSPF adjacencies are FULL |
| `show ip route` | Confirm OSPF routes (marked with `O`) and connected networks |
| `show interfaces tunnel 0` | Check that the tunnel is up and its source and destination |
| `show access-lists` | View ACL rules and match counters |
| `ping <destination-ip>` | Test connectivity between devices |
| `tracert <destination-ip>` | Trace the path between sites |

**Results**

- HQ, SP, and BRANCH exchange routes through OSPF, and each LAN can reach the others.
- Laptops in VLAN 10 and VLAN 20 communicate through the HQ sub-interfaces.
- HQ and BRANCH reach each other over the tunnel (`10.1.1.1` and `10.1.1.2`).
- The ACL blocks 192.168.2.2 and 192.168.2.3, while 192.168.2.4 can still communicate.

---

## Screenshots

<table>
  <tr>
    <td align="center">
      <img width="1308" height="642" alt="Full network topology" src="https://github.com/user-attachments/assets/3b2dffbb-ef16-48ff-9ace-baa8c00ed608" />
      <br/><sub><b>1. Network Topology</b></sub>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img width="658" height="237" alt="VLAN table and trunk status" src="https://github.com/user-attachments/assets/380530bb-e23d-4bd1-ba59-a0b34fcfa6d6" />
      <br/><sub><b>2. VLANs and Trunk (show vlan brief, show interfaces trunk)</b></sub>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img width="741" height="87" alt="PASTE_OSPF_LINK" src="https://github.com/user-attachments/assets/406386fd-9408-4faa-899a-f761e63604b9" />
      <img width="702" height="100" alt="PASTE_OSPF_LINK" src="https://github.com/user-attachments/assets/552a80f1-0f0b-4c4a-8ffa-f05f1570f4a3" />
      <img width="692" height="93" alt="PASTE_OSPF_LINK" src="https://github.com/user-attachments/assets/ccde9486-583c-427a-adfe-2337c3b6e21d" />
      <br/><sub><b>3. OSPF Neighbors and Routes</b></sub>
    </td>
  </tr>
  </table>

---

## Repository Structure

```
OSPFv2-InterVLAN-GRE-ACL/
├── OSPFv2-InterVLAN-GRE-ACL.pkt    # Packet Tracer project file
└── README.md
```

---

## How to Run

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer).
2. Download or clone this repository.
3. Open `OSPFv2-InterVLAN-GRE-ACL.pkt` in Packet Tracer.
4. Open the CLI on each router and run `show ip route`, `show ip ospf neighbor`, and `show interfaces tunnel 0`.
5. Use the Laptop Command Prompt to run `ping` and `tracert` tests.

---

## Skills Demonstrated

- IPv4 Addressing and Subnetting (including /30 point-to-point links)
- VLANs, Access Ports, and Trunking
- Inter-VLAN Routing (Router-on-a-Stick)
- Dynamic Routing with OSPFv2 and Wildcard Masks
- GRE Tunneling
- Standard Access Control Lists
- Cisco IOS CLI
- ICMP Ping and Network Troubleshooting
- Network Documentation

---

## Author

<table>
  <tr>
    <td align="center">
      <a href="https://github.com/mohamedmousa2003">
        <img src="https://github.com/mohamedmousa2003.png" width="90" alt="Mohamed Moussa"/><br/>
        <sub><b>Mohamed Moussa</b></sub>
      </a>
    </td>
  </tr>
</table>

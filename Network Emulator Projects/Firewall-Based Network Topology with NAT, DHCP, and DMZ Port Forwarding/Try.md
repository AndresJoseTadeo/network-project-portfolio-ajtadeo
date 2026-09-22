# Firewall-Based Network Topology with NAT, DHCP, and DMZ Port Forwarding

## Project Overview

This project implements a segmented network topology using a FortiGate firewall as the central security and network services device.

The topology consists of two internal LAN networks, a dedicated management network, an isolated DMZ network containing multiple servers, and an Internet/WAN connection.

The FortiGate firewall provides the following primary services:

- Stateful firewalling
- DHCP
- Source NAT (SNAT)
- Destination NAT (DNAT) / Port Forwarding
- Network segmentation
- Access control between internal, DMZ, management, and external networks

The DMZ contains three servers that are accessible from the Internet through controlled port-forwarding rules. Each server uses a different external TCP port while continuing to provide SSH access internally on TCP port 22.

---

## Devices Used

| Device | Image | Type |
|:---:|:---:|:---:|
| Firewall | fortinet-FGT-v7.0.3build0237 | Qemu |
| Router | i86bi_linux_l2-adventerprisek9-ms.SSA.high_iron_20190423 | IOL |
| Switch | i86bi_LinuxL3-AdvEnterpriseK9-M2_157_3_May_2018 | IOL |


---

## Network Topology

![Network Topology](https://github.com/user-attachments/assets/d8c69eda-f68a-482d-acf5-4b67739718a3)

### Topology Components

| Device | Role |
|:---|:---|
| FortiGate v7.0.3 | Central firewall, DHCP server, NAT gateway, router, and port-forwarding device |
| Router1 | Routes traffic between Internal LAN 1 and the FortiGate |
| Router2 | Routes traffic between Internal LAN 2 and the FortiGate |
| Router_DMZ | Routes traffic between the FortiGate and DMZ network |
| Switch1 | Connects PC-1 and PC-2 |
| Switch2 | Connects PC-3 and PC-4 |
| Switch_DMZ | Connects SRVR1, SRVR2, and SRVR3 |
| PC-1 / PC-2 | Hosts on Internal LAN 1 |
| PC-3 / PC-4 | Hosts on Internal LAN 2 |
| Management PC | Management host connected directly to the FortiGate |
| SRVR1 | DMZ server accessible through TCP port 2221 |
| SRVR2 | DMZ server accessible through TCP port 2222 |
| SRVR3 | DMZ server accessible through TCP port 2223 |

---

# Requirements

The following requirements are implemented in the topology:

- FortiGate firewall as the central security gateway
- Two separate internal networks
- Dedicated management network
- Dedicated DMZ network
- DHCP service for internal hosts
- Internet connectivity through NAT
- Static routing between the FortiGate and downstream routers
- Port forwarding from the Internet to DMZ servers
- SSH access to DMZ servers
- Firewall policies controlling traffic between network segments
- Network isolation between the DMZ and internal networks
- Management access to network infrastructure

---

# IP Addressing

## Network Addressing Scheme

| Network | Subnet | Purpose |
|:---|:---|:---|
| WAN | `100.1.1.0/30` | FortiGate Internet/WAN connection |
| Admin/External | `11.11.11.0/30` | External administration/testing network |
| Management | `192.168.220.0/24` | Network management |
| Internal LAN 1 | `192.168.10.0/24` | PC-1 and PC-2 |
| Internal LAN 2 | `192.168.20.0/24` | PC-3 and PC-4 |
| FortiGate-Router1 | `10.10.10.0/30` | Transit network to Router1 |
| FortiGate-Router2 | `10.10.20.0/30` | Transit network to Router2 |
| FortiGate-Router_DMZ | `10.10.100.0/30` | Transit network to DMZ router |
| DMZ | `172.16.100.0/28` | DMZ servers |

---

## Interface Addressing

| Device | Interface | IP Address | Purpose |
|:---|:---|:---|:---|
| FortiGate | WAN | `100.1.1.2/30` | Internet connection |
| FortiGate | Management | `192.168.220.1/24` | Management network gateway |
| FortiGate | Internal/R1 | `10.10.10.1/30` | Router1 transit gateway |
| FortiGate | Internal/R2 | `10.10.20.1/30` | Router2 transit gateway |
| FortiGate | DMZ | `10.10.100.1/30` | DMZ router transit gateway |
| Router1 | WAN | `10.10.10.2/30` | Connection to FortiGate |
| Router1 | LAN | `192.168.10.1/24` | Internal LAN 1 gateway |
| Router2 | WAN | `10.10.20.2/30` | Connection to FortiGate |
| Router2 | LAN | `192.168.20.1/24` | Internal LAN 2 gateway |
| Router_DMZ | WAN | `10.10.100.2/30` | Connection to FortiGate |
| Router_DMZ | LAN | `172.16.100.1/28` | DMZ gateway |
| SRVR1 | Ethernet | `172.16.100.5/28` | DMZ SSH server |
| SRVR2 | Ethernet | `172.16.100.6/28` | DMZ SSH server |
| SRVR3 | Ethernet | `172.16.100.7/28` | DMZ SSH server |

> **Note:** The `.1` and `.2` interface assignments above are a proposed addressing convention based on the subnetworks shown in the topology. If different addresses were configured in the actual lab, those values should be substituted here.

---

# Loopback Addresses

No loopback interfaces are currently used in this topology.

| Device | Interface | IP Address | Purpose |
|:---|:---|:---|:---|
| N/A | N/A | N/A | No loopback addresses configured |

---

# Technologies Used

- FortiGate Firewall
- FortiOS 7.0.3
- IPv4
- Ethernet
- Static Routing
- DHCP
- Source NAT (SNAT)
- Destination NAT (DNAT)
- Port Forwarding
- Stateful Firewall Policies
- Network Segmentation
- DMZ
- SSH
- TCP/IP
- Private IPv4 Addressing
- `/30` point-to-point networks
- `/24` internal LAN networks
- `/28` DMZ network

---

# Device Configuration

## FortiGate Configuration

The FortiGate acts as the central device connecting the Internet, management network, internal networks, and DMZ.

### FortiGate Network Interfaces

| Interface | Network | Address |
|:---|:---|:---|
| WAN | `100.1.1.0/30` | `100.1.1.2/30` |
| Management | `192.168.220.0/24` | `192.168.220.1/24` |
| Internal 1 | `10.10.10.0/30` | `10.10.10.1/30` |
| Internal 2 | `10.10.20.0/30` | `10.10.20.1/30` |
| DMZ | `10.10.100.0/30` | `10.10.100.1/30` |

---

## Static Routing

The FortiGate requires routes to the networks located behind Router1, Router2, and Router_DMZ.

| Destination | Next Hop | Purpose |
|:---|:---|:---|
| `192.168.10.0/24` | `10.10.10.2` | Internal LAN 1 |
| `192.168.20.0/24` | `10.10.20.2` | Internal LAN 2 |
| `172.16.100.0/28` | `10.10.100.2` | DMZ |
| `0.0.0.0/0` | `100.1.1.1` | Internet default route |

Router1, Router2, and Router_DMZ should use the FortiGate as their default gateway for traffic leaving their respective networks.

---

# Local Network Configuration

## Internal LAN 1

```text
Network:        192.168.10.0/24
Gateway:        192.168.10.1
DHCP Server:    FortiGate
Hosts:          PC-1, PC-2

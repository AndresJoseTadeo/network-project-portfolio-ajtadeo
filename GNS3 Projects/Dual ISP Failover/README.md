## Dual ISP Failover & Load Distribution Network (ON PROGRESS)

## Project Overview
This project implements a small enterprise network with:
  -	Two user VLANs 
  -	Layer 3 inter-VLAN routing 
  -	Centralized DHCP 
  - OSPF between the Layer 3 switch and edge router 
  - Dual Internet Service Providers 
  -	NAT overload/PAT 
  -	IP SLA-based WAN monitoring 
  -	Tracked default routes 
  -	Automatic ISP failover 
  -	VLAN-based Internet traffic distribution

The design uses MLSW as the internal Layer 3 switch and R1 as the Internet edge router.
The two ISP connections provide redundancy so that Internet connectivity can continue if either ISP becomes unavailable.

## Devices Used
| Device | image |
|:---:|:---:|
| Switches L2/L3 | i86bi_linux_l2-adventerprisek9-ms.SSA.high_iron_20190423 |
| Router | c7200-advipservicesk9-mz.152-4.S5 |

## Network Topology
<img width="1264" height="579" alt="image" src="https://github.com/user-attachments/assets/7d5824e4-ab96-445f-bc34-21987f290166" />


## VLAN Addressing

|VLAN	| Network	| Gateway	| DHCP Server |
|:---:|:---:|:---:|:---:|
|VLAN 10	| `192.168.10.0/24`	| `192.168.10.1` | R1 |
|VLAN 20	| `192.168.20.0/24`	| `192.168.20.1` | R1 |


## IP Addressing Table

| Device | Interface | IP Address | Subnet Mask | Purpose |
|:---:|:---:|:---:|:---:|:---:|
| **MLSW** | E0/0 | `10.10.10.1` | `255.255.255.252` | OSPF link to R1 |
| **MLSW** | Lo0 | `10.255.255.1` | `255.255.255.255` | OSPF Router ID |
| **MLSW** | VLAN 10 | `192.168.10.1` | `255.255.255.0` | VLAN 10 Gateway |
| **MLSW** | VLAN 20 | `192.168.20.1` | `255.255.255.0` | VLAN 20 Gateway |
| **R1** | F0/0 | `10.10.10.2` | `255.255.255.252` | OSPF link to MLSW |
| **R1** | G1/0 | `100.1.1.1` | `255.255.255.252` | ISP-A WAN |
| **R1** | G2/0 | `200.1.1.1` | `255.255.255.252` | ISP-B WAN |


## Layer 2 Design
VLAN 10 – Switch1 <br> <br>
<img width="478" height="309" alt="image" src="https://github.com/user-attachments/assets/bd55e75b-50c1-42de-9b46-19c76ddf3caa" /> <br>

<br>

**Switch1 provides Layer 2 connectivity for the VLAN 10 clients.**
Access ports: 
- e0/1 → PC1
- e0/2 → PC2

```text
!Switch1 CONFIG:
enable 
configure terminal
	vlan 10
exit

spanning-tree mode rapid-pvst
interface range e0/1 , e0/2
	switchport mode access
	switchport access vlan 10
	spanning-tree portfast edge

interface e0/0
	switchport trunk encapsulation dot1q
	switchport mode trunk 
	switchport trunk allowed vlan 10
exit
```

<br>

**Switch2 provides Layer 2 connectivity for the VLAN 20 clients.**
Access ports: 
- e0/1 → PC3
- e0/2 → PC4


```text
!Switch2 CONFIG:
enable 
configure terminal
	vlan 20
exit

spanning-tree mode rapid-pvst
interface range e0/1 , e0/2
	switchport mode access
	switchport access vlan 20
	spanning-tree portfast edge

interface e0/0
	switchport trunk encapsulation dot1q
	switchport mode trunk 
	switchport trunk allowed vlan 20
exit
```
<br>

**Multi-Layer Switch**
- MLSW performs the Layer 3 functions for the internal network.
- Inter-VLAN routing 
- Default gateways 
- OSPF adjacency with R1 
- DHCP relay 
- VLAN routing
































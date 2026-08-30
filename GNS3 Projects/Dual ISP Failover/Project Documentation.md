## Dual ISP Failover & Load Distribution Network (IN PROGRESS)

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

## Requirements:

- VLAN 10 `192.168.10.0/24` should use ISP-A as its primary Internet path and ISP-B as its backup path.
- VLAN 20 `192.168.20.0/24` should use ISP-B as its primary Internet path and ISP-A as its backup path.
- Automatic ISP failover should occur when the respective primary ISP connection becomes unavailable.
- R1 should provide DHCP services for both VLAN 10 and VLAN 20.
- R1 should provide Internet connectivity to both VLANs through the two ISP uplinks.
- OSPF should be used between the MLSW and R1 for internal route exchange over the `10.10.10.0/30` link.


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

**VLAN 10 – Switch1** 
<br><br>
<img width="1439" height="732" alt="image" src="https://github.com/user-attachments/assets/3936b37e-36a9-4d33-bb68-5b2e1d27a1f0" />
<br> <br>

**Switch1 provides Layer 2 connectivity for the VLAN 10 clients.** 
<br>

```text
!Switch1 CONFIG:
enable 
configure terminal
	vlan 10
exit
spanning-tree mode rapid-pvst

!Both ports are configured as access ports in VLAN 10.
interface range e0/1 , e0/2
	switchport mode access
	switchport access vlan 10
	spanning-tree portfast edge

!The uplink toward MLSW is configured as an 802.1Q trunk:
interface e0/0
	switchport trunk encapsulation dot1q
	switchport mode trunk 
	switchport trunk allowed vlan 10
exit
```

<br>

**VLAN 20 – Switch2** 
<br><br>
<img width="1462" height="720" alt="image" src="https://github.com/user-attachments/assets/738103ea-5550-4da8-a9bf-5377ae6528ed" />
<br><br>

**Switch2 provides Layer 2 connectivity for the VLAN 20 clients.**
<br>

```text
!Switch2 CONFIG:
enable 
configure terminal
	vlan 20
exit
spanning-tree mode rapid-pvst

!Both ports are configured as access ports in VLAN 20.
interface range e0/1 , e0/2
	switchport mode access
	switchport access vlan 20
	spanning-tree portfast edge

!The uplink toward MLSW is configured as an 802.1Q trunk:
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

```text
!MLSW CONFIG:
enable
configure terminal
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20 priority 4096
ip routing
vlan 10
vlan 20
exit

!Links to L2 switches
interface e0/1
	no shutdown
	switchport trunk encapsulation dot1q 
	switchport mode trunk 
	switchport trunk allowed vlan 10
interface e0/2
	no shutdown
	switchport trunk encapsulation dot1q 
	switchport mode trunk 
	switchport trunk allowed vlan 20
exit

interface e0/0
	no switchport 
	no shutdown
	ip address 10.10.10.1 255.255.255.252
	ip ospf 1 area 0 
	ip ospf network point-to-point
interface loopback 0
	ip address 10.255.255.1 255.255.255.255
	ip ospf 1 area 0

!SVIs as gateways for VLANs
interface vlan 10
	ip address 192.168.10.1 255.255.255.0
	ip ospf 1 area 0
	no shutdown
interface vlan 20
	ip address 192.168.20.1 255.255.255.0
	ip ospf 1 area 0
	no shutdown
exit

!OSPF 
router ospf 1
	router-id 10.255.255.1 
	passive-interface vlan 10
	passive-interface vlan 20
	passive-interface loopback 0
exit
```































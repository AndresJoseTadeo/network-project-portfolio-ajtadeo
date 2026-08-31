
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
    
I designed the network to use MLSW as the internal Layer 3 switch and R1 as the Internet edge router.
The two ISP connections provide redundancy so that Internet connectivity can continue if either ISP becomes unavailable.

## Devices Used
| Device | image |
|:---:|:---:|
| Switches L2/L3 | i86bi_linux_l2-adventerprisek9-ms.SSA.high_iron_20190423 |
| Router | c7200-advipservicesk9-mz.152-4.S5 |

## Network Topology
<img width="1264" height="579" alt="image" src="https://github.com/user-attachments/assets/7d5824e4-ab96-445f-bc34-21987f290166" />

### Topology Components

| Device | Role |
|:---:|:---:|
| Switch1 | Access switch for VLAN 10 |
| Switch2 | Access switch for VLAN 20 |
| MLSW | Multilayer switch / Layer 3 gateway |
| R1 | Edge router, DHCP server, NAT/PAT, ISP failover |
| ISP-A | Primary ISP for VLAN 10 |
| ISP-B | Primary ISP for VLAN 20 |
| PC1 / PC2 | VLAN 10 clients |
| PC3 / PC4 | VLAN 20 clients |

---

## Requirements:

- VLAN 10 `192.168.10.0/24` should use ISP-A as its primary Internet path and ISP-B as its backup path.
- VLAN 20 `192.168.20.0/24` should use ISP-B as its primary Internet path and ISP-A as its backup path.
- If the primary ISP becomes unavailable, IP SLA should detect the failure, and object tracking must allow traffic to switch to the backup ISP.
- R1 should provide DHCP services for both VLAN 10 and VLAN 20.
- R1 should provide Internet connectivity to both VLANs through the two ISP uplinks.
- OSPF should be used between the MLSW and R1 for internal route exchange over the `10.10.10.0/30` link.

| VLAN | Network | Primary ISP | Backup ISP |
|:---:|:---:|:---:|:---:|
| VLAN 10 | `192.168.10.0/24` | ISP-A | ISP-B |
| VLAN 20 | `192.168.20.0/24` | ISP-B | ISP-A |

---

# IP Addressing

## VLAN Addressing

| VLAN | Network | Default Gateway | DHCP Server |
|:---:|:---:|:---:|:---:|
| VLAN 10 | `192.168.10.0/24` | `192.168.10.1` | R1 |
| VLAN 20 | `192.168.20.0/24` | `192.168.20.1` | R1 |

## MLSW-to-R1 Link

| Device | Interface | IP Address |
|:---:|:---:|:---:|
| MLSW | `E0/0` | `10.10.10.1/30` |
| R1 | `F0/0` | `10.10.10.2/30` |

## ISP-A Link

| Device | Interface | IP Address |
|:---:|:---:|:---:|
| R1 | `G1/0` | `100.1.1.1/30` |
| ISP-A | `G1/0` | `100.1.1.2/30` |

## ISP-B Link

| Device | Interface | IP Address |
|:---:|:---:|:---:|
| R1 | `G2/0` | `200.1.1.1/30` |
| ISP-B | `G1/0` | `200.1.1.2/30` |

## Loopback Addresses

| Device | Interface | IP Address | Purpose |
|:---:|:---:|:---:|:---:|
| MLSW | Loopback0 | `10.255.255.1/32` | OSPF Router ID |
| R1 | Loopback0 | `10.255.255.2/32` | OSPF Router ID |
| ISP-A | Loopback0 | `10.255.255.10/32` | IP SLA target |
| ISP-B | Loopback0 | `10.255.255.20/32` | IP SLA target |
| ISP-A | Loopback10 | `8.8.8.8/32` | Simulated Internet |
| ISP-B | Loopback10 | `8.8.8.8/32` | Simulated Internet |

---

# Traffic Failover Design

The intended ISP path for each VLAN is:

### VLAN 10

<img width="1048" height="296" alt="image" src="https://github.com/user-attachments/assets/dd38e2e6-153a-45c1-82f2-e34f1b1c2b8f" />

### VLAN 20

<img width="1049" height="297" alt="image" src="https://github.com/user-attachments/assets/364e5461-6a7b-401d-a880-046f41f621be" /> <br>

| Source Network | Primary Path | Backup Path |
|:---:|:---:|:---:|
| `192.168.10.0/24` | ISP-A | ISP-B |
| `192.168.20.0/24` | ISP-B | ISP-A |

---

# Technologies Used

## Layer 2 – Switching
- VLANs
- 802.1Q Trunking
- Rapid-PVST

## Layer 3 – Routing & Switching
- Layer 3 Switching
- Inter-VLAN Routing
- OSPF
- Static Routing
  
## Network Services
- DHCP
- DHCP Relay

## WAN / Failover
- IP SLA
- Object Tracking
- Route Maps
- NAT/PAT
- Dual ISP Connectivity

---

# Device Configuration

## Switch1

Switch1 provides Layer 2 connectivity for the VLAN 10 clients.

The uplink toward the MLSW operates as an 802.1Q trunk.

```cisco
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

---

## Switch2

Switch2 provides Layer 2 connectivity for the VLAN 20 clients.

The uplink toward the MLSW operates as an 802.1Q trunk.

```cisco
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

---

## Multilayer Switch (MLSW)

The MLSW performs:

- VLAN creation
- Inter-VLAN routing
- Default gateway services
- OSPF
- DHCP relay
- Trunk connectivity toward the access switches
- Layer 3 connectivity toward R1

## MLSW Configuration

```cisco
enable
configure terminal

spanning-tree mode rapid-pvst
spanning-tree vlan 10,20 priority 4096

ip routing

vlan 10
vlan 20
exit

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

interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 ip ospf 1 area 0
 no shutdown

interface vlan 20
 ip address 192.168.20.1 255.255.255.0
 ip ospf 1 area 0
 no shutdown

exit

router ospf 1
 router-id 10.255.255.1
 passive-interface vlan 10
 passive-interface vlan 20
 passive-interface loopback 0

exit
```

---

# DHCP Configuration

- R1 provides centralized DHCP services for both VLANs.
- The MLSW uses DHCP relay to forward client DHCP requests to R1.

## MLSW DHCP Relay

The DHCP requests are forwarded to R1's Loopback0 address `10.255.255.2`

```cisco
interface vlan 10
 ip helper-address 10.255.255.2

interface vlan 20
 ip helper-address 10.255.255.2
```
---
# R1 Configuration

R1 performs the following functions:

- OSPF routing
- DHCP server
- WAN connectivity
- IP SLA monitoring
- Object tracking
- Default route failover
- NAT/PAT
- Route-map based NAT

---

## R1 Internal Interface

The connection between R1 and the MLSW uses the `10.10.10.0/30` network.

```cisco
interface f0/0
 no shutdown
 ip address 10.10.10.2 255.255.255.252
 ip ospf 1 area 0
 ip ospf network point-to-point
 ip nat inside
```

---

# R1 OSPF Configuration

R1 uses Loopback0 as its OSPF Router ID.

```cisco
interface loopback 0
 ip address 10.255.255.2 255.255.255.255
 ip ospf 1 area 0

exit

router ospf 1
 router-id 10.255.255.2
 passive-interface loopback 0
 default-information originate

exit
```

R1 originates the default route into OSPF so internal networks can reach external networks through R1.

---

# R1 DHCP Configuration

R1 provides DHCP addresses for both VLANs.

```cisco
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10

ip dhcp pool VLAN-10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1

exit

ip dhcp pool VLAN-20
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1

exit
```

---

# WAN Configuration

R1 has two independent WAN connections.

```text
R1 G1/0 → ISP-A
R1 G2/0 → ISP-B
```

## ISP-A Interface

```cisco
interface g1/0
 ip address 100.1.1.1 255.255.255.252
 no shutdown
 ip nat outside
```

## ISP-B Interface

```cisco
interface g2/0
 ip address 200.1.1.1 255.255.255.252
 no shutdown
 ip nat outside
```

## Internal NAT Interface

```cisco
interface f0/0
 ip nat inside

exit
```

---

# ISP-A Configuration

ISP-A uses `100.1.1.2/30` on its connection to R1.

Loopback0 is used as the IP SLA monitoring target, while Loopback10 provides the simulated Internet destination.

```cisco
enable
configure terminal

interface g1/0
 no shutdown
 ip address 100.1.1.2 255.255.255.252

exit

interface loopback 0
 ip address 10.255.255.10 255.255.255.255

exit

interface loopback 10
 ip address 8.8.8.8 255.255.255.255

exit
```

---

# ISP-B Configuration

ISP-B uses `200.1.1.2/30` on its connection to R1.

Loopback0 is used as the IP SLA monitoring target, while Loopback10 provides the simulated Internet destination.

```cisco
enable
configure terminal

interface g1/0
 no shutdown
 ip address 200.1.1.2 255.255.255.252

exit

interface loopback 0
 ip address 10.255.255.20 255.255.255.255

exit

interface loopback 10
 ip address 8.8.8.8 255.255.255.255

exit
```

---

# IP SLA Monitoring

IP SLA monitors the availability of both ISP connections.

R1 sends ICMP echo requests toward the monitoring address of each ISP using the corresponding WAN interface as the source.

## ISP-A IP SLA

```cisco
ip sla 10
 icmp-echo 10.255.255.10 source-interface g1/0
 threshold 50
 frequency 5
 timeout 100

exit

ip sla schedule 10 start-time now life forever
```

## ISP-B IP SLA

```cisco
ip sla 20
 icmp-echo 10.255.255.20 source-interface g2/0
 threshold 50
 frequency 5
 timeout 100

exit

ip sla schedule 20 start-time now life forever
```

---

# Object Tracking

The IP SLA operations are associated with tracking objects.

```cisco
track 10 ip sla 10 reachability
track 20 ip sla 20 reachability
```

The tracking objects monitor the reachability of the ISP paths.

When an ISP becomes unavailable, the corresponding tracking object changes state and the associated tracked route is removed.

---

# ISP Monitoring Routes

R1 uses static host routes to reach the IP SLA monitoring addresses through their respective ISPs.

## ISP-A Monitoring Route

```cisco
ip route 10.255.255.10 255.255.255.255 g1/0 100.1.1.2
```

## ISP-B Monitoring Route

```cisco
ip route 10.255.255.20 255.255.255.255 g2/0 200.1.1.2
```

---

# Default Route Failover

The default routes are associated with the IP SLA tracking objects.

## ISP-A

```cisco
ip route 0.0.0.0 0.0.0.0 g1/0 100.1.1.2 track 10
```

## ISP-B

```cisco
ip route 0.0.0.0 0.0.0.0 g2/0 200.1.1.2 track 20
```

The tracking mechanism allows R1 to automatically remove a default route when the corresponding ISP becomes unavailable.

---

# NAT/PAT Configuration

Private addresses from VLAN 10 and VLAN 20 are translated before accessing the Internet.

## NAT Access Lists

```cisco
access-list 10 permit 192.168.10.0 0.0.0.255
access-list 20 permit 192.168.20.0 0.0.0.255

exit
```

---

# Primary NAT Route Maps

## VLAN 10 → ISP-A

```cisco
route-map ISP-A permit 10
 match ip address 10
 match interface g1/0

exit

ip nat inside source route-map ISP-A interface g1/0 overload
```

## VLAN 20 → ISP-B

```cisco
route-map ISP-B permit 10
 match ip address 20
 match interface g2/0

exit

ip nat inside source route-map ISP-B interface g2/0 overload
```

---

# Failover NAT Route Maps

Additional NAT route maps provide translation through the backup ISP.

## VLAN 10 → ISP-B During Failover

```cisco
route-map ISP-A_FAILOVER permit 10
 match ip address 10
 match interface g2/0

exit

ip nat inside source route-map ISP-A_FAILOVER interface g2/0 overload
```

## VLAN 20 → ISP-A During Failover

```cisco
route-map ISP-B_FAILOVER permit 10
 match ip address 20
 match interface g1/0

exit

ip nat inside source route-map ISP-B_FAILOVER interface g1/0 overload
```

---

# Complete R1 Configuration

The complete R1 configuration is provided below for reference.

```cisco
enable
configure terminal

!
! ==============================
! INTERNAL OSPF INTERFACE
! ==============================
!

interface f0/0
 no shutdown
 ip address 10.10.10.2 255.255.255.252
 ip ospf 1 area 0
 ip ospf network point-to-point
 ip nat inside

interface loopback 0
 ip address 10.255.255.2 255.255.255.255
 ip ospf 1 area 0

router ospf 1
 router-id 10.255.255.2
 passive-interface loopback 0
 default-information originate

!
! ==============================
! DHCP
! ==============================
!

ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10

ip dhcp pool VLAN-10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1

exit

ip dhcp pool VLAN-20
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1

exit

!
! ==============================
! ISP-A
! ==============================
!

interface g1/0
 ip address 100.1.1.1 255.255.255.252
 no shutdown
 ip nat outside

!
! ==============================
! ISP-B
! ==============================
!

interface g2/0
 ip address 200.1.1.1 255.255.255.252
 no shutdown
 ip nat outside

!
! ==============================
! IP SLA
! ==============================
!

ip sla 10
 icmp-echo 10.255.255.10 source-interface g1/0
 threshold 50
 frequency 5
 timeout 100

exit

ip sla 20
 icmp-echo 10.255.255.20 source-interface g2/0
 threshold 50
 frequency 5
 timeout 100

exit

ip sla schedule 10 start-time now life forever
ip sla schedule 20 start-time now life forever

!
! ==============================
! OBJECT TRACKING
! ==============================
!

track 10 ip sla 10 reachability
track 20 ip sla 20 reachability

!
! ==============================
! ISP MONITORING ROUTES
! ==============================
!

ip route 10.255.255.10 255.255.255.255 g1/0 100.1.1.2
ip route 10.255.255.20 255.255.255.255 g2/0 200.1.1.2

!
! ==============================
! DEFAULT ROUTES
! ==============================
!

ip route 0.0.0.0 0.0.0.0 g1/0 100.1.1.2 track 10
ip route 0.0.0.0 0.0.0.0 g2/0 200.1.1.2 track 20

!
! ==============================
! NAT ACCESS LISTS
! ==============================
!

access-list 10 permit 192.168.10.0 0.0.0.255
access-list 20 permit 192.168.20.0 0.0.0.255

exit

!
! ==============================
! PRIMARY NAT - ISP-A
! ==============================
!

route-map ISP-A permit 10
 match ip address 10
 match interface g1/0

exit

ip nat inside source route-map ISP-A interface g1/0 overload

!
! ==============================
! PRIMARY NAT - ISP-B
! ==============================
!

route-map ISP-B permit 10
 match ip address 20
 match interface g2/0

exit

ip nat inside source route-map ISP-B interface g2/0 overload

!
! ==============================
! FAILOVER NAT - VLAN 10
! ==============================
!

route-map ISP-A_FAILOVER permit 10
 match ip address 10
 match interface g2/0

exit

ip nat inside source route-map ISP-A_FAILOVER interface g2/0 overload

!
! ==============================
! FAILOVER NAT - VLAN 20
! ==============================
!

route-map ISP-B_FAILOVER permit 10
 match ip address 20
 match interface g1/0

exit

ip nat inside source route-map ISP-B_FAILOVER interface g1/0 overload

exit
```

---

# Verification

The following commands can be used to verify the operation of the network.

## Verify VLANs

```cisco
show vlan brief
```

## Verify Trunking

```cisco
show interfaces trunk
```

## Verify Interfaces

```cisco
show ip interface brief
```

## Verify OSPF Neighbor

```cisco
show ip ospf neighbor
```

## Verify OSPF Routes

```cisco
show ip route ospf
```

## Verify DHCP

```cisco
show ip dhcp binding
show ip dhcp pool
```

## Verify IP SLA

```cisco
show ip sla statistics
```

## Verify Object Tracking

```cisco
show track
```

## Verify Routing Table

```cisco
show ip route
```

## Verify NAT Translations

```cisco
show ip nat translations
```

## Verify NAT Statistics

```cisco
show ip nat statistics
```

---

# Failover Testing

## Test 1 — ISP-A Failure

Under normal conditions, VLAN 10 uses ISP-A as its primary path.

```text
VLAN 10
   ↓
MLSW
   ↓
R1
   ↓
ISP-A
   ↓
Internet
```

After ISP-A becomes unavailable:

```text
VLAN 10
   ↓
MLSW
   ↓
R1
   ↓
ISP-B
   ↓
Internet
```

### Verification

```cisco
show ip sla statistics
show track
show ip route
show ip nat translations
```

---

## Test 2 — ISP-B Failure

Under normal conditions, VLAN 20 uses ISP-B as its primary path.

```text
VLAN 20
   ↓
MLSW
   ↓
R1
   ↓
ISP-B
   ↓
Internet
```

After ISP-B becomes unavailable:

```text
VLAN 20
   ↓
MLSW
   ↓
R1
   ↓
ISP-A
   ↓
Internet
```

### Verification

```cisco
show ip sla statistics
show track
show ip route
show ip nat translations
```

---

# Connectivity Testing

The following tests can be performed from the PCs.

## VLAN 10

```text
ping 192.168.10.1
ping 10.10.10.2
ping 8.8.8.8
```

## VLAN 20

```text
ping 192.168.20.1
ping 10.10.10.2
ping 8.8.8.8
```

Internet connectivity should remain available after failure of the respective primary ISP.

---

# Expected Failover Behavior

| Test | Expected Primary | Expected Backup |
|---|---|---|
| VLAN 10 — Normal | ISP-A | ISP-B |
| VLAN 10 — ISP-A failure | ISP-B | — |
| VLAN 20 — Normal | ISP-B | ISP-A |
| VLAN 20 — ISP-B failure | ISP-A | — |

---

# Project Results

The completed network provides:

- VLAN-based network segmentation
- Inter-VLAN routing through the MLSW
- Centralized DHCP services through R1
- OSPF-based internal route exchange
- Dual ISP connectivity
- NAT/PAT for private networks
- ISP health monitoring using IP SLA
- Object tracking for route failover
- Primary and backup ISP paths for each VLAN
- Automatic ISP failover
- Continued Internet connectivity during an ISP failure

---

# Skills Demonstrated

- VLAN Configuration
- 802.1Q Trunking
- Rapid-PVST
- Layer 3 Switching
- Inter-VLAN Routing
- OSPF
- DHCP
- DHCP Relay
- Static Routing
- IP SLA
- Object Tracking
- Route Maps
- NAT/PAT
- Dual ISP Connectivity
- Automatic WAN Failover
- Network Troubleshooting
- Network Verification

---

# Repository Structure

```text
Dual-ISP-Failover/
│
├── README.md
│
├── images/
│   └── dual-isp-failover.png
│
└── configs/
    ├── Switch1.txt
    ├── Switch2.txt
    ├── MLSW.txt
    ├── R1.txt
    ├── ISP-A.txt
    └── ISP-B.txt
```

---

# Conclusion

This project demonstrates the implementation of a redundant dual-ISP network using Cisco routing and switching technologies.

The design provides separate primary ISP paths for VLAN 10 and VLAN 20 while maintaining a backup path for each network. IP SLA and object tracking monitor ISP availability and allow the network to automatically respond to WAN failures.

The combination of VLAN segmentation, OSPF, centralized DHCP, NAT/PAT, route maps, IP SLA, and object tracking provides a practical example of enterprise WAN redundancy and automatic failover.

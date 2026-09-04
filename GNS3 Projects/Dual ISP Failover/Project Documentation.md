<img width="1396" height="987" alt="image" src="https://github.com/user-attachments/assets/b44e5002-3ac4-4c80-b8e4-50aafd823d2d" /># DUAL ISP FAILOVER (WORK IN PROGRESS)

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

---

## Devices Used
| Device | image |
|:---:|:---:|
| Switches L2/L3 | i86bi_linux_l2-adventerprisek9-ms.SSA.high_iron_20190423 |
| Router | c7200-advipservicesk9-mz.152-4.S5 |

---

## Network Topology

<img width="2044" height="979" alt="image" src="https://github.com/user-attachments/assets/84ac9a14-82de-4657-9c18-14d15ce6d181" />


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

<img width="1680" height="470" alt="image" src="https://github.com/user-attachments/assets/fad86960-e132-45d4-b6ab-dd7a5a55bac1" />



### VLAN 20

<img width="1680" height="470" alt="image" src="https://github.com/user-attachments/assets/93da94c0-51f3-48c1-9f3d-89359960bdc4" /> <br>

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

## Local Network Configuration

<img width="1089" height="754" alt="image" src="https://github.com/user-attachments/assets/168a7e97-811c-4ab1-a9e0-52d25f17c106" />

## 1. Switch1

**Switch1 provides Layer 2 connectivity for the VLAN 10 clients.**

a) Creating VLAN 10 and enabling Rapid-PVST+

```cisco
enable
configure terminal

vlan 10
exit

spanning-tree mode rapid-pvst
```

b) Assigning interfaces as VLAN 10 access ports

```cisco
interface range e0/1 , e0/2
  switchport mode access
  switchport access vlan 10
  spanning-tree portfast edge
```

c) Configuring the uplink toward the MLSW as an 802.1Q trunk
```cisco
interface e0/0
  switchport trunk encapsulation dot1q
  switchport mode trunk
  switchport trunk allowed vlan 10

exit
```
---

## 2. Switch2

**Switch2 provides Layer 2 connectivity for the VLAN 20 clients.**

a) Creating VLAN 20 and enabling Rapid-PVST+

```cisco
enable
configure terminal

vlan 20
exit

spanning-tree mode rapid-pvst
```

b) Assigning interfaces as VLAN 20 access ports

```cisco
interface range e0/1 , e0/2
  switchport mode access
  switchport access vlan 20
  spanning-tree portfast edge
```

c) Configuring the uplink toward the MLSW as an 802.1Q trunk
```cisco
interface e0/0
  switchport trunk encapsulation dot1q
  switchport mode trunk
  switchport trunk allowed vlan 20

exit
```
---

## 3. Multilayer Switch (MLSW)

**The MLSW performs:**
- VLAN creation
- Inter-VLAN routing
- Default gateway services
- OSPF
- DHCP relay
- Trunk connectivity toward the access switches
- Layer 3 connectivity toward R1

a) Enabling Layer 3 routing and configuring Rapid-PVST+
```cisco
enable
configure terminal

spanning-tree mode rapid-pvst
spanning-tree vlan 10,20 priority 4096

ip routing
```

b) Creating VLAN 10 and VLAN 20
```cisco
vlan 10
vlan 20
exit
```

c) Configuring the trunk toward Switch1
```cisco
interface e0/1
  no shutdown
  switchport trunk encapsulation dot1q
  switchport mode trunk
  switchport trunk allowed vlan 10
```

d) Configuring the trunk toward Switch2
```cisco
interface e0/2
  no shutdown
  switchport trunk encapsulation dot1q
  switchport mode trunk
  switchport trunk allowed vlan 20
```

e) Configuring the Layer 3 link toward R1
```cisco
interface e0/0
  no switchport
  no shutdown
  ip address 10.10.10.1 255.255.255.252
  ip ospf 1 area 0
  ip ospf network point-to-point
```

f) Configuring Loopback0 for OSPF
```cisco
interface loopback 0
  ip address 10.255.255.1 255.255.255.255
  ip ospf 1 area 0
```

g) Configuring VLAN 10 SVI and DHCP relay
```cisco
interface vlan 10
  ip address 192.168.10.1 255.255.255.0
  ip ospf 1 area 0
  ip helper-address 10.255.255.2
  no shutdown
```

h) Configuring VLAN 20 SVI and DHCP relay
```cisco
interface vlan 20
  ip address 192.168.20.1 255.255.255.0
  ip ospf 1 area 0
  ip helper-address 10.255.255.2
  no shutdown
```

i) Configuring OSPF
```cisco
router ospf 1
  router-id 10.255.255.1
  passive-interface vlan 10
  passive-interface vlan 20
  passive-interface loopback 0

exit
```

---

## 4. R1 (Router)

**R1 performs:**
- Centralized DHCP services for both VLANs.
- OSPF routing
- DHCP server
- WAN connectivity
- IP SLA monitoring
- Object tracking
- Default route failover
- NAT/PAT
- Route-map based NAT

<br>

a) Configuring the internal link toward the MLSW
```cisco
enable
configure terminal

interface f0/0
  no shutdown
  ip address 10.10.10.2 255.255.255.252
  ip ospf 1 area 0
  ip ospf network point-to-point
  ip nat inside
```

b) Configuring Loopback0 for OSPF and DHCP relay reachability
```cisco
interface loopback 0
  ip address 10.255.255.2 255.255.255.255
  ip ospf 1 area 0
```

c) Configuring OSPF
```cisco
router ospf 1
  router-id 10.255.255.2
  passive-interface loopback 0
  default-information originate always
```

d) Configuring DHCP 

- DHCP exclusions
```cisco
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
```

- VLAN 10 DHCP pool
```cisco
ip dhcp pool VLAN-10
  network 192.168.10.0 255.255.255.0
  default-router 192.168.10.1
```

- VLAN 20 DHCP pool
```cisco
ip dhcp pool VLAN-20
  network 192.168.20.0 255.255.255.0
  default-router 192.168.20.1

exit
```

---

# WAN Configuration

<img width="902" height="574" alt="image" src="https://github.com/user-attachments/assets/311fc7f1-e762-491a-809d-ae2e4b639b81" />


## 1. R1 (Router)
**R1 provides dual-WAN connectivity through ISP-A and ISP-B.**
- R1 G1/0 connects to ISP-A using the `100.1.1.0/30` network.
- R1 G2/0 connects to ISP-B using the `200.1.1.0/30` network.
- R1 F0/0 connects to the MLSW 
  
a) Configuring the WAN interface toward ISP-A
```cisco
enable
configure terminal

interface g1/0
  ip address 100.1.1.1 255.255.255.252
  ip nat outside
  no shutdown
```

b) Configuring the WAN interface toward ISP-B
```cisco
interface g2/0
  ip address 200.1.1.1 255.255.255.252
  ip nat outside
  no shutdown
exit
```

## 2. ISP-A Configuration

**ISP-A uses:**
- `100.1.1.2/30` for connection to R1
- `10.255.255.10/32` as the IP SLA monitoring address
- `8.8.8.8/32` as the simulated Internet destination.

a) Configuring the WAN interface
```cisco
enable
configure terminal

interface g1/0
  ip address 100.1.1.2 255.255.255.252
  no shutdown

exit
```

b) Configuring Loopback0 for IP SLA monitoring
```cisco
interface loopback 0
 ip address 10.255.255.10 255.255.255.255

exit
```

c) Configuring Loopback10 as the simulated Internet destination
```cisco
interface loopback 10
 ip address 8.8.8.8 255.255.255.255

exit
```

## 3. ISP-B Configuration

**ISP-B uses:**
- `200.1.1.2/30` for connection to R1
- `10.255.255.20/32` as the IP SLA monitoring address
- `8.8.8.8/32` as the simulated Internet destination.

a) Configuring the WAN interface
```cisco
enable
configure terminal

interface g1/0
  ip address 200.1.1.2 255.255.255.252
  no shutdown

exit
```

b) Configuring Loopback0 for IP SLA monitoring
```cisco
interface loopback 0
 ip address 10.255.255.20 255.255.255.255

exit
```

c) Configuring Loopback10 as the simulated Internet destination
```cisco
interface loopback 10
 ip address 8.8.8.8 255.255.255.255

exit
```

# IP SLA Configuration
**R1 uses IP SLA to continuously monitor the availability of both ISP connections.**

R1 sends ICMP echo requests toward each ISP's monitoring address.

<img width="815" height="549" alt="image" src="https://github.com/user-attachments/assets/246a42b8-71ce-4826-93db-c64f9a8cc2b8" />

## IP SLA 10

a) Configuring the ISP-A monitoring route
```cisco
ip route 10.255.255.10 255.255.255.255 g1/0 100.1.1.2
```

b) Configuring IP SLA for ISP-A
```cisco
ip sla 10
 icmp-echo 10.255.255.10 source-interface g1/0
 threshold 50
 frequency 5
 timeout 100

exit
```

c) Scheduling IP SLA 10
```cisco
ip sla schedule 10 start-time now life forever
```

## IP SLA 20

a) Configuring the ISP-B monitoring route
```cisco
ip route 10.255.255.20 255.255.255.255 g2/0 200.1.1.2
```

b) Configuring IP SLA for ISP-B
```cisco
ip sla 20
 icmp-echo 10.255.255.20 source-interface g2/0
 threshold 50
 frequency 5
 timeout 100

exit
```

c) Scheduling IP SLA 20
```cisco
ip sla schedule 20 start-time now life forever
```

<br>

**R1 sends an ICMP echo request to each ISP every 5 seconds.** 


- R1 g1/0 <--> ISP-A g1/0
<img width="2002" height="613" alt="image" src="https://github.com/user-attachments/assets/85afa357-b6bc-4bdd-bf83-750b3cd24b61" />

<br><br>

- R1 g2/0 <--> ISP-b g1/0
<img width="1994" height="612" alt="image" src="https://github.com/user-attachments/assets/e57175da-b987-4d5a-bf22-47ef6b68412a" />

---

# Object Tracking 
**Object tracking links the IP SLA results to the WAN default routes.**
When an IP SLA operation fails, its corresponding tracking object changes state.

a) Tracking ISP-A
```cisco
track 10 ip sla 10 reachability
```

b) Tracking ISP-B
```cisco
track 20 ip sla 20 reachability
```

<img width="344" height="323" alt="image" src="https://github.com/user-attachments/assets/34eb9ee3-60a6-4e43-b3a7-16175e5fa2e6" />

# Default Route Failover
**R1 uses tracked static default routes to provide automatic WAN failover.**
When the corresponding IP SLA becomes unreachable, the tracked default route is removed.

a) Configuring the ISP-A default route (tracking ISP-A)
```cisco
ip route 0.0.0.0 0.0.0.0 g1/0 100.1.1.2 track 10
```

b) Configuring the ISP-B default route (tracking ISP-B)
```cisco
ip route 0.0.0.0 0.0.0.0 g2/0 200.1.1.2 track 20
```
<img width="897" height="587" alt="image" src="https://github.com/user-attachments/assets/099f31b8-9fac-427a-a432-73fbce752e57" />

# NAT/PAT Configuration
**R1 performs PAT for the private VLAN networks before traffic is sent to the Internet.**

a) Configuring the VLAN 10 NAT access list
```cisco
access-list 10 permit 192.168.10.0 0.0.0.255
```

b) Configuring the VLAN 20 NAT access list
```cisco
access-list 20 permit 192.168.20.0 0.0.0.255
```

## Primary NAT Route Maps
**These Route maps determine which WAN interface is used for NAT based on the source VLAN and outgoing interface.**

a) VLAN 10 ==> ISP-A
- Match the outgoing interface to ISP-A
- Match IP address in ACL 10.
- Permit sequence 10 in ACL 10.

```cisco
route-map ISP-A permit 10
 match ip address 10
 match interface g1/0

exit

ip nat inside source route-map ISP-A interface g1/0 overload
```

b) VLAN 20 ==> ISP-B
- Match the outgoing interface to ISP-B
- Match IP address in ACL 20.
- Permit sequence 10 in ACL 20.
  
```cisco
route-map ISP-B permit 10
 match ip address 20
 match interface g2/0

exit

ip nat inside source route-map ISP-B interface g2/0 overload
```

## Failover NAT Route Maps
**These additional route maps allow the VLANs to be translated through the alternate ISP when traffic is routed through the backup WAN interface.**

a) VLAN 10 ==> ISP-B during ISP-A failure
- Match the outgoing interface to ISP-B
- Match IP address in ACL 10.
- Permit sequence 10 in ACL 10.
  
```cisco
route-map ISP-A_FAILOVER permit 10
 match ip address 10
 match interface g2/0

exit

ip nat inside source route-map ISP-A_FAILOVER interface g2/0 overload
```

b) VLAN 20 ==> ISP-A during ISP-B failure
- Match the outgoing interface to ISP-A
- Match IP address in ACL 20.
- Permit sequence 10 in ACL 20.
  
```cisco
route-map ISP-B_FAILOVER permit 10
 match ip address 20
 match interface g1/0

exit

ip nat inside source route-map ISP-B_FAILOVER interface g1/0 overload
```


# TEST RESULTS, FAILOVER SIMULATION (WIP)

## Scenario 1: Both ISPs are working properly
<img width="1772" height="849" alt="image" src="https://github.com/user-attachments/assets/f267cf88-d837-4e92-beca-cf274233e601" />

a) Ping / Trace 8.8.8.8 (VLAN 10)
<img width="1396" height="987" alt="image" src="https://github.com/user-attachments/assets/3e467b17-7955-46f2-b8fa-7af9015c3f3d" />

b) Ping / Trace 8.8.8.8 (VLAN 20)
<img width="1397" height="981" alt="image" src="https://github.com/user-attachments/assets/94440c5c-e625-4aca-a005-241c83fe5370" />

c) IP SLA / Tracking Status
<img width="1398" height="1297" alt="image" src="https://github.com/user-attachments/assets/b0bf55f9-bc36-48fd-b191-9ffe8ecba8fe" />

## Scenario 2 : Link to ISP-A fails




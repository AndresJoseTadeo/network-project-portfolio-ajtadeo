# Basic Firewall Network Topology

## Project Overview

In this project, I designed and configured a **firewall-based network topology using a FortiGate Firewall** as the main security device.

The main goal of this project was to build a network where I could practice and demonstrate several important networking concepts, including:

- Firewall policies
- DHCP
- OSPF / Static routing
- Source NAT (SNAT)
- Destination NAT (DNAT) with Port Forwarding
- DMZ network segmentation
- Remote server administration

I divided the network into an **Internal Network**, a **DMZ Network**, and an **External/Internet Network**. This allows me to control how traffic moves between different parts of the network instead of allowing unrestricted communication.

The FortiGate acts as the central point for firewall policies, NAT, DHCP, and traffic control.

---

# Network Topology

<img width="1522" height="1080" alt="image" src="https://github.com/user-attachments/assets/18d0e70d-5a08-40e2-8706-8b5e8f519e48" />

---

# Network Topology Components

| Device | Type | Role |
|:---|:---|:---|
| FortiGate | Firewall | Main firewall, NAT, DHCP, and security gateway |
| Router1 | Router | Connects LAN 1 to the FortiGate |
| Router2 | Router | Connects LAN 2 to the FortiGate |
| Router_DMZ | Router | Connects the DMZ to the FortiGate |
| Switch1 | Switch | LAN 1 access switch |
| Switch2 | Switch | LAN 2 access switch |
| Switch_DMZ | Switch | DMZ access switch |
| PC-1 | PC | LAN 1 client |
| PC-2 | PC | LAN 1 client |
| PC-3 | PC | LAN 2 client |
| PC-4 | PC | LAN 2 client |
| SRVR1 | Server | DMZ SSH server |
| SRVR2 | Server | DMZ SSH server |
| SRVR3 | Server | DMZ SSH server |
| Admin | PC/Workstation | Remote administrator |
| Internet | Network | External network |
| VMware Network | Virtual Network | Management network (allows me to access FortiGate's GUI) |

---

## Topology Overview

The topology is divided into several sections.

### Internal Network

The Internal Network contains two LANs:

- **LAN 1:** `192.168.10.0/24`
- **LAN 2:** `192.168.20.0/24`

LAN 1 contains PC-1 and PC-2, while LAN 2 contains PC-3 and PC-4.

### DMZ Network

The DMZ contains three servers:

- **SRVR1:** `172.16.100.5`
- **SRVR2:** `172.16.100.6`
- **SRVR3:** `172.16.100.7`

The servers are intended to provide SSH access only. SSH uses the standard TCP port `22`.

### External Network

The public address used for SNAT,  DNAT and port forwarding configuration is:

```text
100.1.1.1
```

### Management Network

I also have a VMware management network:

```text
192.168.222.0/24
```

This network connection allows me to access the Firewall's GUI and make configurations.

---

# Project Requirements

For this project, I wanted the network to meet several requirements.

## 1. Communication Between LAN 1 and LAN 2

The first requirement is to allow communication between the two internal LANs.

```text
LAN 1
192.168.10.0/24
       |
       v
    Router1
       |
       v
   FortiGate
       |
       v
    Router2
       |
       v
LAN 2
192.168.20.0/24
```

I configured the network so that devices in LAN 1 can communicate with devices in LAN 2 and vice versa.

For example, PC-1 should be able to reach a device in LAN 2:

```bash
ping 192.168.20.x
```

Similarly, a device in LAN 2 should be able to reach a device in LAN 1.

---

# 2. DHCP for the Internal Networks

Instead of manually assigning IP addresses to every PC, I configured the FortiGate to provide DHCP services for the internal networks.

For LAN 1:

```text
Network: 192.168.10.0/24
Gateway: 192.168.10.1
DHCP:    FortiGate
```

For LAN 2:

```text
Network: 192.168.20.0/24
Gateway: 192.168.20.1
DHCP:    FortiGate
```

This allows PC-1, PC-2, PC-3, and PC-4 to automatically receive their IP configuration.

The DHCP service provides the clients with:

- IP address
- Subnet mask
- Default gateway
- DNS information

---

# 3. DMZ Network Segmentation

One of the main parts of this project is the DMZ.

I separated the servers from the Internal Network by placing them in a dedicated DMZ:

```text
DMZ Network
172.16.100.0/28
```

The servers are:

| Server | IP Address | Service |
|:---|:---:|:---|
| SRVR1 | `172.16.100.5` | SSH |
| SRVR2 | `172.16.100.6` | SSH |
| SRVR3 | `172.16.100.7` | SSH |

The servers use SSH on the standard TCP port `22`.

The reason I placed the servers in a DMZ is to create a separation between the servers and the internal users.

The general traffic flow I want is:

```text
Internal Network
       |
       | ALLOWED
       v
      DMZ
```

But I do not want the servers to freely initiate new connections toward the Internal Network:

```text
DMZ
 |
 | NEW CONNECTION
 v
FortiGate
 |
 X
 |
Internal Network
```

This gives me better control over the traffic between the two zones.

---

# 4. Internal Network to DMZ

I configured the firewall so that the Internal Network can initiate connections toward the DMZ.

For example, an administrator inside the Internal Network can connect to SRVR1:

```bash
ssh user@172.16.100.5
```

The traffic follows this general path:

```text
Internal PC
     |
     v
FortiGate
     |
     v
DMZ
     |
     v
SRVR1
```

The same concept applies to SRVR2 and SRVR3.

---

# 5. DMZ to Internal Network

For security purposes, I configured the firewall so that DMZ servers cannot initiate new connections toward the Internal Network.

For example, if SRVR1 tries to start a new connection toward a PC in LAN 1:

```text
SRVR1
172.16.100.5
     |
     v
FortiGate
     |
     X
     |
192.168.10.0/24
```

the connection should be denied.

The intended behavior is:

```text
Internal ---> DMZ
   ALLOWED

DMZ ---> Internal
   DENIED
```

This still allows return traffic for connections that were originally initiated from the Internal Network.

---

# 6. Source NAT (SNAT)

I configured the FortiGate to perform **Source NAT (SNAT)** for the Internal Network when accessing the Internet.

The internal PCs use private IP addresses such as:

```text
192.168.10.x
192.168.20.x
```

These addresses are translated by the FortiGate when the traffic leaves the network.

The traffic flow looks like this:

```text
PC-1
192.168.10.10
     |
     v
FortiGate
     |
     | SNAT
     v
Public IP
100.1.1.1
     |
     v
Internet
```

This allows the internal hosts to access external networks without directly exposing their private addresses.

---

# 7. Destination NAT (DNAT) and Virtual IPs

Another important part of my configuration is **Destination NAT (DNAT)**.

I used FortiGate Virtual IPs (VIPs) to forward incoming connections from the public address to the appropriate DMZ server.

The servers use SSH on the standard TCP port `22`.

However, I assigned a different public port to each server.

| Server | Public Address | Public Port | Private Address | Private Port |
|:---|:---:|:---:|:---:|:---:|
| SRVR1 | `100.1.1.1` | `2221` | `172.16.100.5` | `22` |
| SRVR2 | `100.1.1.1` | `2222` | `172.16.100.6` | `22` |
| SRVR3 | `100.1.1.1` | `2223` | `172.16.100.7` | `22` |

---

# SSH Access to the Servers

The three servers are used for SSH access only. I kept the standard SSH port, which is port `22`, on each server.

Instead of giving each server a different public IP address, I configured different ports on the FortiGate. The FortiGate then sends the connection to the correct server.

For example, when the administrator connects to:

```text
100.1.1.1:2221
```

the FortiGate forwards the connection to Server 1 on its SSH port:

```text
172.16.100.5:22
```

The same setup is used for the other two servers:

```text
100.1.1.1:2221 -> Server 1
100.1.1.1:2222 -> Server 2
100.1.1.1:2223 -> Server 3
```

The ports used in this project are mainly for demonstration. In a real-world network, the ports would depend on the service being provided. For example, web services commonly use port `443` for HTTPS, while SSH commonly uses port `22`.

The public port and the port used by the server do not always have to be the same. The FortiGate can receive traffic on one port and forward it to the appropriate port on the server.

This gives the administrator more flexibility when setting up access to different services.

---

# Remote Administration

The **Admin** workstation represents a remote administrator who needs to manage the servers.

Instead of connecting directly to the private IP addresses, the administrator connects to the public address:

```text
100.1.1.1
```

and specifies the appropriate port.

For example:

```bash
ssh user@100.1.1.1 -p 2221
```

The FortiGate then translates the destination to:

```text
172.16.100.5:22
```

The same concept is used for the other servers.

This allows me to demonstrate how port forwarding can be used to provide controlled remote access to services inside a private network.

---

# 8. OSPF for Internal Routing

I used **OSPF (Open Shortest Path First)** for internal routing.

The purpose of using OSPF is to allow the routers to dynamically exchange routing information instead of manually configuring every internal route.

The main internal networks are:

```text
192.168.10.0/24
192.168.20.0/24
172.16.100.0/28
```

The transit networks are:

```text
10.10.10.0/30
10.10.20.0/30
10.10.100.0/30
```

The basic internal routing design looks like this:

```text
                FortiGate
                /       \
               /         \
          Router1       Router2
             |             |
             |             |
           LAN 1         LAN 2
```

I use OSPF to exchange the internal routes between the routers.

---

# 9. Static Routing for External Connectivity

For external routing, I use static routing.

The general idea is for the FortiGate to have a default route toward the Internet.

```text
Internal Network
       |
       v
    FortiGate
       |
       | Default Route
       v
Internet Gateway
       |
       v
    Internet
```

This keeps the internal routing dynamic through OSPF while using a simple static route for external connectivity.

---

# IP Addressing

## Network Summary

| Network | Purpose |
|:---|:---|
| `100.1.1.0/30` | External/Internet network |
| `10.10.10.0/30` | FortiGate ↔ Router1 |
| `10.10.20.0/30` | FortiGate ↔ Router2 |
| `10.10.100.0/30` | FortiGate ↔ Router_DMZ |
| `192.168.10.0/24` | LAN 1 |
| `192.168.20.0/24` | LAN 2 |
| `172.16.100.0/28` | DMZ |
| `192.168.222.0/24` | VMware Management Network |

---

# External Network

```text
Network:       100.1.1.0/30
Public IP:     100.1.1.1
```

| Device | Interface | IP Address |
|:---|:---|:---:|
| FortiGate | WAN | `TBD` |
| Internet Gateway | External | `100.1.1.1` |

> I will update the FortiGate WAN address based on the final lab configuration.

---

# FortiGate to Router1

```text
Network: 10.10.10.0/30
```

| Device | Interface | IP Address |
|:---|:---|:---:|
| FortiGate | Internal Interface | `10.10.10.1/30` |
| Router1 | e0/1 | `10.10.10.2/30` |

---

# FortiGate to Router2

```text
Network: 10.10.20.0/30
```

| Device | Interface | IP Address |
|:---|:---|:---:|
| FortiGate | Internal Interface | `10.10.20.1/30` |
| Router2 | e0/1 | `10.10.20.2/30` |

---

# FortiGate to Router_DMZ

```text
Network: 10.10.100.0/30
```

| Device | Interface | IP Address |
|:---|:---|:---:|
| FortiGate | DMZ Interface | `10.10.100.1/30` |
| Router_DMZ | e0/1 | `10.10.100.2/30` |

---

# LAN 1 Addressing

```text
Network: 192.168.10.0/24
```

| Device | IP Address | Assignment |
|:---|:---:|:---|
| Router1 | `192.168.10.1/24` | Static |
| PC-1 | `DHCP` | Dynamic |
| PC-2 | `DHCP` | Dynamic |

---

# LAN 2 Addressing

```text
Network: 192.168.20.0/24
```

| Device | IP Address | Assignment |
|:---|:---:|:---|
| Router2 | `192.168.20.1/24` | Static |
| PC-3 | `DHCP` | Dynamic |
| PC-4 | `DHCP` | Dynamic |

---

# DMZ Addressing

```text
Network: 172.16.100.0/28
```

| Device | IP Address | Service |
|:---|:---:|:---|
| Router_DMZ | `172.16.100.1/28` | DMZ Gateway |
| SRVR1 | `172.16.100.5/28` | SSH |
| SRVR2 | `172.16.100.6/28` | SSH |
| SRVR3 | `172.16.100.7/28` | SSH |

All servers use SSH on:

```text
TCP/22
```

---

# VMware Management Network

```text
Network: 192.168.222.0/24
```

| Device | IP Address |
|:---|:---:|
| Management Workstation | `192.168.222.108` |
| FortiGate Management | `192.168.222.141` |

I use this network mainly for management and access to the virtualized environment.

---

# Loopback Addresses

I can use loopback interfaces on the routers for OSPF router identification.

| Device | Interface | IP Address | Purpose |
|:---|:---|:---|:---|
| Router1 | Loopback0 | `TBD` | OSPF Router ID |
| Router2 | Loopback0 | `TBD` | OSPF Router ID |
| Router_DMZ | Loopback0 | `TBD` | OSPF Router ID |

> These addresses can be updated once the final router configuration is completed.

---

# Technologies Used

The main technologies I used in this project are:

- FortiGate Firewall
- Firewall Policies
- DHCP
- OSPF
- Static Routing
- Source NAT (SNAT)
- Destination NAT (DNAT)
- Virtual IPs (VIPs)
- DMZ Segmentation
- SSH
- IPv4
- Ethernet Switching
- VMware Virtual Networking

---

# Firewall Security Policies

I designed the firewall policies around the required traffic flows.

| # | Source | Destination | Service | Action | Purpose |
|:---:|:---|:---|:---|:---:|:---|
| 1 | LAN 1 | LAN 2 | Required Services | ALLOW | Internal communication |
| 2 | LAN 2 | LAN 1 | Required Services | ALLOW | Internal communication |
| 3 | Internal | DMZ | SSH/Required | ALLOW | Server management |
| 4 | DMZ | Internal | New Connections | DENY | DMZ isolation |
| 5 | Internal | Internet | Required Services | ALLOW + SNAT | Internet access |
| 6 | Internet | SRVR1 | TCP/2221 | ALLOW + DNAT | SRVR1 SSH |
| 7 | Internet | SRVR2 | TCP/2222 | ALLOW + DNAT | SRVR2 SSH |
| 8 | Internet | SRVR3 | TCP/2223 | ALLOW + DNAT | SRVR3 SSH |
| 9 | Internet | Internal | Any | DENY | Protect internal network |

The main idea is to only allow the traffic that I actually need.

---

# NAT Configuration

## Source NAT

For Internet access, the internal private addresses are translated by the FortiGate.

```text
192.168.10.0/24
        |
        |
192.168.20.0/24
        |
        v
   +----------+
   | FortiGate|
   +----------+
        |
        | SNAT
        v
   100.1.1.1
        |
        v
     Internet
```

---

# Destination NAT

For incoming SSH connections, I configured Virtual IPs.

## VIP 1 - SRVR1

```text
External Address: 100.1.1.1
External Port:    2221

Internal Address: 172.16.100.5
Internal Port:    22

Protocol: TCP
```

---

## VIP 2 - SRVR2

```text
External Address: 100.1.1.1
External Port:    2222

Internal Address: 172.16.100.6
Internal Port:    22

Protocol: TCP
```

---

## VIP 3 - SRVR3

```text
External Address: 100.1.1.1
External Port:    2223

Internal Address: 172.16.100.7
Internal Port:    22

Protocol: TCP
```

---

# Device Configuration

## Local Network Configuration

### LAN 1

```text
Network:  192.168.10.0/24
Gateway:  192.168.10.1
DHCP:     FortiGate
```

Topology:

```text
             Router1
                |
                |
             Switch1
             /     \
            /       \
          PC-1      PC-2
```

---

### LAN 2

```text
Network:  192.168.20.0/24
Gateway:  192.168.20.1
DHCP:     FortiGate
```

Topology:

```text
             Router2
                |
                |
             Switch2
             /     \
            /       \
          PC-3      PC-4
```

---

### DMZ

```text
Network:  172.16.100.0/28
Gateway:  172.16.100.1
```

Topology:

```text
             Router_DMZ
                 |
                 |
             Switch_DMZ
             /    |    \
            /     |     \
         SRVR1  SRVR2  SRVR3
```

---

# OSPF Configuration

I use OSPF to advertise the internal networks between the routers.

The networks involved are:

```text
10.10.10.0/30
10.10.20.0/30
10.10.100.0/30

192.168.10.0/24
192.168.20.0/24
172.16.100.0/28
```

For example, a Cisco-style configuration for Router1 could look like:

```cisco
router ospf 1
 router-id 1.1.1.1
 network 10.10.10.0 0.0.0.3 area 0
 network 192.168.10.0 0.0.0.255 area 0
```

For Router2:

```cisco
router ospf 1
 router-id 2.2.2.2
 network 10.10.20.0 0.0.0.3 area 0
 network 192.168.20.0 0.0.0.255 area 0
```

These are examples and I will adjust them based on the actual interfaces and IP addresses in the final configuration.

---

# Static Routing

For external connectivity, I use a default static route.

The basic design is:

```text
Internal Network
       |
       v
    Router
       |
       v
   FortiGate
       |
       v
Internet Gateway
       |
       v
    Internet
```

The FortiGate handles the connection between the internal network and the external network.

---

# DHCP Configuration

I configured the FortiGate to provide DHCP addresses for the two internal LANs.

## LAN 1 DHCP Scope

```text
Network:
192.168.10.0/24

Default Gateway:
192.168.10.1

DHCP:
Enabled
```

Example DHCP range:

```text
192.168.10.100 - 192.168.10.200
```

---

## LAN 2 DHCP Scope

```text
Network:
192.168.20.0/24

Default Gateway:
192.168.20.1

DHCP:
Enabled
```

Example DHCP range:

```text
192.168.20.100 - 192.168.20.200
```

> The actual DHCP ranges can be changed depending on the final FortiGate configuration.

---

# Test Results

After configuring the topology, I would verify each major requirement separately.

---

## 1. DHCP Test

First, I check whether the PCs receive their IP addresses automatically.

Expected:

```text
PC-1 -> 192.168.10.x
PC-2 -> 192.168.10.x

PC-3 -> 192.168.20.x
PC-4 -> 192.168.20.x
```

On Windows:

```cmd
ipconfig
```

On Linux:

```bash
ip addr
```

Expected result:

```text
DHCP SUCCESS
```

---

# 2. LAN 1 to LAN 2 Connectivity

From PC-1:

```bash
ping 192.168.20.x
```

Expected result:

```text
SUCCESS
```

From PC-3:

```bash
ping 192.168.10.x
```

Expected result:

```text
SUCCESS
```

This confirms that communication between the two internal LANs is working.

---

# 3. Internal to DMZ Connectivity

From an internal PC, I can test connectivity to the DMZ servers:

```bash
ping 172.16.100.5
ping 172.16.100.6
ping 172.16.100.7
```

I can also test SSH:

```bash
ssh user@172.16.100.5
ssh user@172.16.100.6
ssh user@172.16.100.7
```

Expected:

```text
Internal -> DMZ
ALLOWED
```

---

# 4. DMZ to Internal Connectivity

Next, I test whether the DMZ servers can initiate connections toward the Internal Network.

For example:

```bash
ping 192.168.10.x
```

Expected result:

```text
DENIED
```

This confirms that the DMZ isolation policy is working.

---

# 5. Internet Connectivity

From an internal PC:

```bash
ping 8.8.8.8
```

Expected:

```text
SUCCESS
```

If this works, I can verify that the FortiGate is performing SNAT for the connection.

---

# 6. SSH Port Forwarding Test

## SRVR1

From the Admin workstation:

```bash
ssh user@100.1.1.1 -p 2221
```

Expected traffic flow:

```text
100.1.1.1:2221
        |
        v
FortiGate
        |
       DNAT
        |
        v
172.16.100.5:22
        |
        v
SRVR1
```

Expected result:

```text
SSH CONNECTION SUCCESS
```

---

## SRVR2

```bash
ssh user@100.1.1.1 -p 2222
```

Expected traffic flow:

```text
100.1.1.1:2222
        |
        v
FortiGate
        |
       DNAT
        |
        v
172.16.100.6:22
        |
        v
SRVR2
```

Expected result:

```text
SSH CONNECTION SUCCESS
```

---

## SRVR3

```bash
ssh user@100.1.1.1 -p 2223
```

Expected traffic flow:

```text
100.1.1.1:2223
        |
        v
FortiGate
        |
       DNAT
        |
        v
172.16.100.7:22
        |
        v
SRVR3
```

Expected result:

```text
SSH CONNECTION SUCCESS
```

---

# 7. OSPF Verification

I can verify the OSPF neighbors using:

```bash
show ip ospf neighbor
```

I can also check the routes learned through OSPF:

```bash
show ip route ospf
```

Expected:

```text
OSPF Neighbor:
UP
```

and the appropriate internal routes should appear in the routing table.

---

# 8. Routing Table Verification

I can check the routing table with:

```bash
show ip route
```

I should be able to see routes for:

```text
192.168.10.0/24
192.168.20.0/24
172.16.100.0/28
```

along with the appropriate transit and external routes.

---

# 9. NAT Verification

I can also verify that the FortiGate is translating the internal traffic.

The expected behavior is:

```text
Private Source
192.168.10.x
       |
       v
   FortiGate
       |
      SNAT
       |
       v
Public Source
100.1.1.1
```

For the incoming SSH connections, the expected DNAT behavior is:

```text
100.1.1.1:2221 ---> 172.16.100.5:22

100.1.1.1:2222 ---> 172.16.100.6:22

100.1.1.1:2223 ---> 172.16.100.7:22
```

---

# Test Matrix

| Test | Source | Destination | Expected Result |
|:---|:---|:---|:---|
| DHCP | PC-1 | DHCP Server | PASS |
| DHCP | PC-2 | DHCP Server | PASS |
| DHCP | PC-3 | DHCP Server | PASS |
| DHCP | PC-4 | DHCP Server | PASS |
| LAN communication | LAN 1 | LAN 2 | ALLOW |
| LAN communication | LAN 2 | LAN 1 | ALLOW |
| Internal to DMZ | Internal | SRVR1 | ALLOW |
| Internal to DMZ | Internal | SRVR2 | ALLOW |
| Internal to DMZ | Internal | SRVR3 | ALLOW |
| DMZ isolation | SRVR1 | LAN 1 | DENY |
| DMZ isolation | SRVR2 | LAN 2 | DENY |
| Internet access | Internal | Internet | ALLOW + SNAT |
| SSH forwarding | Admin | `100.1.1.1:2221` | DNAT to SRVR1 |
| SSH forwarding | Admin | `100.1.1.1:2222` | DNAT to SRVR2 |
| SSH forwarding | Admin | `100.1.1.1:2223` | DNAT to SRVR3 |
| OSPF | Router1/Router2 | OSPF Neighbor | UP |
| Routing | Internal | External | Reachable |

---

# Observations

While working on this topology, I observed several important networking concepts.

First, using the FortiGate as the central firewall makes it easier to control traffic between the different network zones. Instead of treating the entire network as one flat network, I can create policies that determine exactly which traffic should be allowed.

The separation between the Internal Network and DMZ is also important. The internal users can access the servers when necessary, but the servers cannot simply start new connections toward the internal clients.

I also used DHCP to simplify IP address assignment for the internal PCs. This means I do not have to manually configure every client.

For routing, I used OSPF for the internal network so that the routers can dynamically exchange routes. For external connectivity, I used static routing because the external path is relatively simple.

Another important part of the project is NAT. SNAT allows the internal clients to access the Internet using a public address, while DNAT allows selected external connections to reach specific DMZ servers.

The SSH port forwarding setup was particularly useful because it allowed me to use one public IP address while still reaching three different servers by using different external ports.

The public ports used in this project are mainly for demonstrating how port forwarding works. In a real network, the ports would depend on the service being accessed. For example, HTTPS commonly uses port `443`, while SSH commonly uses port `22`.

---

# Security Considerations

I designed the topology with network segmentation in mind.

The basic architecture is:

```text
                    INTERNET
                        |
                        |
                    100.1.1.1
                        |
                        v
                +---------------+
                |   FortiGate   |
                |    Firewall   |
                +---------------+
                  /           \
                 /             \
                /               \
       INTERNAL NETWORK        DMZ NETWORK
              |                    |
              |                    |
         User Devices          SSH Servers
                                  |
                         +--------+--------+
                         |        |        |
                       SRVR1    SRVR2    SRVR3
```

The Internal Network contains the user devices, while the DMZ contains the servers that need to be accessed.

For external SSH access, I only expose the ports that I specifically configured:

```text
TCP/2221 -> SRVR1:22
TCP/2222 -> SRVR2:22
TCP/2223 -> SRVR3:22
```

Other unsolicited inbound traffic should remain blocked unless there is a specific requirement to allow it.

---

# Overall Traffic Flow

## Internal to Internal

```text
LAN 1
  |
  v
FortiGate
  |
  v
LAN 2
```

**Status:** Allowed

---

## Internal to DMZ

```text
Internal
   |
   v
FortiGate
   |
   v
DMZ
```

**Status:** Allowed

---

## DMZ to Internal

```text
DMZ
 |
 v
FortiGate
 |
 X
 |
Internal
```

**Status:** Denied for new connections

---

## Internal to Internet

```text
Internal
   |
   v
FortiGate
   |
  SNAT
   |
   v
Internet
```

**Status:** Allowed

---

## Internet to SRVR1

```text
Internet
   |
   | 100.1.1.1:2221
   v
FortiGate
   |
  DNAT
   |
   v
172.16.100.5:22
   |
   v
SRVR1
```

**Status:** Allowed

---

## Internet to SRVR2

```text
Internet
   |
   | 100.1.1.1:2222
   v
FortiGate
   |
  DNAT
   |
   v
172.16.100.6:22
   |
   v
SRVR2
```

**Status:** Allowed

---

## Internet to SRVR3

```text
Internet
   |
   | 100.1.1.1:2223
   v
FortiGate
   |
  DNAT
   |
   v
172.16.100.7:22
   |
   v
SRVR3
```

**Status:** Allowed

---

# Complete Addressing Reference

| Network / Device | Address | Description |
|:---|:---|:---|
| Public Address | `100.1.1.1` | External/public address |
| External Network | `100.1.1.0/30` | Internet transit |
| FortiGate ↔ Router1 | `10.10.10.0/30` | LAN 1 transit |
| FortiGate ↔ Router2 | `10.10.20.0/30` | LAN 2 transit |
| FortiGate ↔ Router_DMZ | `10.10.100.0/30` | DMZ transit |
| LAN 1 | `192.168.10.0/24` | Internal LAN 1 |
| LAN 2 | `192.168.20.0/24` | Internal LAN 2 |
| DMZ | `172.16.100.0/28` | Server network |
| SRVR1 | `172.16.100.5` | SSH Server 1 |
| SRVR2 | `172.16.100.6` | SSH Server 2 |
| SRVR3 | `172.16.100.7` | SSH Server 3 |
| VMware Network | `192.168.222.0/24` | Management network |
| Management PC | `192.168.222.108` | Management workstation |
| FortiGate Management | `192.168.222.141` | Firewall management |

---

# SSH Port Mapping Reference

| Server | Private IP | Private Port | Public IP | Public Port |
|:---|:---:|:---:|:---:|:---:|
| SRVR1 | `172.16.100.5` | `22` | `100.1.1.1` | `2221` |
| SRVR2 | `172.16.100.6` | `22` | `100.1.1.1` | `2222` |
| SRVR3 | `172.16.100.7` | `22` | `100.1.1.1` | `2223` |

---

# Conclusion

In this project, I built a firewall-based network using a FortiGate as the main security device.

I configured two internal LANs, a separate DMZ, DHCP services, OSPF routing, static external routing, SNAT, and DNAT. I also configured firewall policies to control how the Internal Network, DMZ, and Internet can communicate with each other.

One of the main things I wanted to demonstrate was the difference between **internal access to the DMZ** and **external access to the DMZ**. Internal hosts are allowed to initiate connections toward the servers, while the servers are prevented from initiating new connections toward the Internal Network.

For remote administration, I configured SSH port forwarding using the public address `100.1.1.1`. Each server uses a different public port:

```text
100.1.1.1:2221 ---> 172.16.100.5:22
100.1.1.1:2222 ---> 172.16.100.6:22
100.1.1.1:2223 ---> 172.16.100.7:22
```

This allows the administrator to manage all three servers remotely while keeping SSH on port 22 on the servers.

The port numbers used for forwarding can be different depending on the service. For example, a web server commonly uses HTTPS on port `443`, while SSH normally uses port `22`. The FortiGate can use a different public port and forward the connection to the port used by the service on the server.

Overall, this project helped me understand how routing, NAT, firewall policies, DHCP, and network segmentation work together in a practical network environment.

---

# Project Status

| Feature | Status |
|:---|:---:|
| Internal LAN 1 | Implemented |
| Internal LAN 2 | Implemented |
| LAN 1 ↔ LAN 2 Communication | Implemented |
| DHCP | Implemented |
| DMZ | Implemented |
| DMZ Isolation | Implemented |
| OSPF | Implemented |
| Static External Routing | Implemented |
| SNAT | Implemented |
| DNAT | Implemented |
| Virtual IPs | Implemented |
| SSH Port Forwarding | Implemented |
| Remote Administration | Implemented |
| Testing | To Be Verified |
| Final Configuration | To Be Updated |

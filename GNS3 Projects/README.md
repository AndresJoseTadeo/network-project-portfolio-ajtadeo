## Dual ISP Failover & Load Distribution Network

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

## Network Topology
<img width="1264" height="579" alt="image" src="https://github.com/user-attachments/assets/7d5824e4-ab96-445f-bc34-21987f290166" />


## IP Addressing Table

| Device | Interface | IP Address | Subnet Mask | Purpose |
|---|---|---|---|---|
| **MLSW** | E0/0 | `10.10.10.1` | `255.255.255.252` | OSPF link to R1 |
| **MLSW** | Lo0 | `10.255.255.1` | `255.255.255.255` | OSPF Router ID |
| **MLSW** | VLAN 10 | `192.168.10.1` | `255.255.255.0` | VLAN 10 Gateway |
| **MLSW** | VLAN 20 | `192.168.20.1` | `255.255.255.0` | VLAN 20 Gateway |
| **R1** | F0/0 | `10.10.10.2` | `255.255.255.252` | OSPF link to MLSW |
| **R1** | G1/0 | `100.1.1.1` | `255.255.255.252` | ISP-A WAN |
| **R1** | G2/0 | `200.1.1.1` | `255.255.255.252` | ISP-B WAN |

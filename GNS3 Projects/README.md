Dual ISP Failover & Load Distribution Network


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

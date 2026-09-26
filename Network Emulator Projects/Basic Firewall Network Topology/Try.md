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

The network is divided into an **Internal Network**, a **DMZ Network**, and an **External/Internet Network**. This allows us to control how traffic moves between different parts of the network instead of allowing unrestricted communication.

The FortiGate acts as the central point for firewall policies, NAT, DHCP, and traffic control.

---

## Network Topology

<img width="1522" height="1080" alt="image" src="https://github.com/user-attachments/assets/18d0e70d-5a08-40e2-8706-8b5e8f519e48" />


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

The public address used for SNAT,  DNAT and port forwarding configuration is: ```100.1.1.1```


### Management Network

I also have a VMware management network: ```192.168.222.0/24```


This network connection allows me to access the Firewall's GUI and make configurations.


---


## Network Topology Components

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
| INTERNET | Router | Simulated Internet |
| VMware Network | Virtual Network | Management network (allows me to access FortiGate's GUI) |

---

## IP Addressing Summary

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


## Initial Configuration

### Switches
#### 1. Switch1
```cisco
enable
config terminal
hostname Switch1
end
```

#### 2. Switch2
```cisco
enable
config terminal
hostname Switch2
end
```

#### 3. Switch_DMZ
```cisco
enable
config terminal
hostname Switch_DMZ
end
```

### Routers
#### 1. Router1
```cisco
enable
config terminal
hostname Router1
int e0/0
	no shutdown
	description To-LAN1
	ip address 192.168.10.1 255.255.255.0
int e0/1
	no shutdown
	description To-Fortigate
	ip address 10.10.10.1 255.255.255.252
end
```

#### 2. Router2
```cisco
enable
config terminal
hostname Router2
int e0/0
	no shutdown
	description To-LAN2
	ip address 192.168.20.1 255.255.255.0
int e0/1
	no shutdown
	description To-Fortigate
	ip address 10.10.20.1 255.255.255.252
end
```

#### 3. Router_DMZ
```cisco
enable
config terminal
hostname Router_DMZ
int e0/0
	no shutdown
	description To-DMZ
	ip address 172.16.100.1 255.255.255.240
int e0/1
	no shutdown
	description To-Fortigate
	ip address 10.10.100.1 255.255.255.252
end
```

#### 4. INTERNET
```cisco
enable
config terminal
hostname INTERNET
int e0/0
	no shutdown
	description WAN-To-Fortigate
	ip address 100.1.1.2 255.255.255.252
int loopback 0
	no shutdown
	description Simulated_Internet
	ip address 8.8.8.8 255.255.255.255
end
```

### FortiGate Firewall Setup
#### a. Initial Configuration

```cisco
username : admin
password : password123


config system global
show
config system global
    set admintimeout 30
    set alias "FortiGate-VM64-KVM"
    set hostname "FortiGate"
    set timezone 04
end

config system interface
edit port1
set mode static
set ip 192.168.222.141/24
set allowaccess https http ping ssh
end
```
*I'm using FortiGate's `port1` as the management interface, connected to the emulator's Cloud0 interface, which provides connectivity to the VMware virtual network. This allows me to access the FortiGate GUI from my PC for initial configuration.*

<img width="711" height="445" alt="image" src="https://github.com/user-attachments/assets/1a46c42c-9386-436f-8fba-a1b533199b54" />

#### FortiGate's login page
<img width="1920" height="1036" alt="image" src="https://github.com/user-attachments/assets/021f49d1-b113-4091-80e5-4219c0e30c9c" />

#### FortiGate's Dashboard
<img width="1920" height="1039" alt="image" src="https://github.com/user-attachments/assets/a3ae628d-d39b-4584-a3d3-cced28f02fbe" />

--- 

## Routing Configuration
<img width="820" height="428" alt="image" src="https://github.com/user-attachments/assets/7e74398c-d526-4aad-b495-5f2a3a6f797b" />


### OSPF (Single Area)
1. Router1
```cisco
enable
configure terminal
router ospf 1
	router-id 10.255.255.255.1
	network 192.168.10.0 0.0.0.255 area 0
	network 10.10.10.0 0.0.0.3 area 0
```
2. Router2
```cisco
enable
configure terminal
router ospf 1
	router-id 10.255.255.255.2
	network 192.168.20.0 0.0.0.255 area 0
	network 10.10.20.0 0.0.0.3 area 0
```
3. Router_DMZ
```cisco
enable
configure terminal
router ospf 1
	router-id 10.255.255.255.4
	network 172.16.100.0 0.0.0.255 area 0
	network 10.10.100.0 0.0.0.3 area 0
```

4. FortiGate
```cisco
config router ospf
    set default-information-originate always
    set router-id 10.255.255.3
    config area
        edit 0.0.0.0
        next
    end
    config ospf-interface
        edit "OSPF-Port2"
            set interface "port2"
            set prefix-length 30
        next
        edit "OSPF-Port3"
            set interface "port3"
            set prefix-length 30
        next
        edit "OSPF-Port4-DMZ"
            set interface "port4"
        next
    end
    config network
        edit 1
            set prefix 10.10.10.0 255.255.255.252
        next
        edit 2
            set prefix 10.10.20.0 255.255.255.252
        next
        edit 3
            set prefix 10.10.100.0 255.255.255.252
        next
    end
    config redistribute "connected"
    end
    config redistribute "static"
        set status enable
        set metric-type 1
    end
```

#### Static


### DHCP Configuration


### Firewall Policies




   


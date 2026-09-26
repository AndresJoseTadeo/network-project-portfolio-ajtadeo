
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
#### 1. Router1
```cisco
enable
configure terminal
router ospf 1
	router-id 10.255.255.255.1
	network 192.168.10.0 0.0.0.255 area 0
	network 10.10.10.0 0.0.0.3 area 0
```
#### 2. Router2
```cisco
enable
configure terminal
router ospf 1
	router-id 10.255.255.255.2
	network 192.168.20.0 0.0.0.255 area 0
	network 10.10.20.0 0.0.0.3 area 0
```
#### 3. Router_DMZ
```cisco
enable
configure terminal
router ospf 1
	router-id 10.255.255.255.4
	network 172.16.100.0 0.0.0.255 area 0
	network 10.10.100.0 0.0.0.3 area 0
```

#### 4. FortiGate
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

### Static Routing (Towards WAN)

#### FortiGate
<img width="526" height="406" alt="image" src="https://github.com/user-attachments/assets/5238e52c-1acb-4495-baf2-ad2c43c59519" />


### Routing Table Verification
<img width="1078" height="706" alt="image" src="https://github.com/user-attachments/assets/ad5aa489-f95b-44bf-9ce5-6799320e0afe" />

--- 

## DHCP Configuration
<img width="716" height="671" alt="image" src="https://github.com/user-attachments/assets/b2038282-713d-4e19-a93a-d7a1952b72ff" />


### DHCP Server
#### Fortigate
```cisco
    edit 2
        set dns-service default
        set default-gateway 192.168.10.1
        set netmask 255.255.255.0
        set interface "port2"
        config ip-range
            edit 1
                set start-ip 192.168.10.10
                set end-ip 192.168.10.254
            next
        end
    next
    edit 3
        set dns-service default
        set default-gateway 192.168.20.1
        set netmask 255.255.255.0
        set interface "port3"
        config ip-range
            edit 1
                set start-ip 192.168.20.10
                set end-ip 192.168.20.254
            next
        end
```

### DHCP Relay Agents
#### Router1
```cisco
enable
configure terminal
int e0/0
	ip helper-address 10.10.10.2
end
```
#### Router2
```cisco
enable
configure terminal
int e0/0
	ip helper-address 10.10.10.2
end
```

### Firewall Policies

#### Internatl-Network 
<img width="997" height="967" alt="image" src="https://github.com/user-attachments/assets/6806b9a9-70ed-4915-af34-5770fe7111ed" />
<img width="998" height="973" alt="image" src="https://github.com/user-attachments/assets/6bf0c070-4c34-4268-b48d-3d44c87296f3" />
<img width="995" height="955" alt="image" src="https://github.com/user-attachments/assets/c4db9d5c-091e-4aa2-9083-cfc3f517b09e" />
<img width="994" height="954" alt="image" src="https://github.com/user-attachments/assets/115cd7c6-ef84-4fdd-9d2d-9900bbcf2fba" />
<img width="997" height="953" alt="image" src="https://github.com/user-attachments/assets/f7ad5064-ea10-4b54-94ea-0e426ae1a8b7" />



#### DNAT / Port Forwarding
<img width="1217" height="157" alt="image" src="https://github.com/user-attachments/assets/77bf27bf-6e8c-480d-87de-724926c0648f" />

<img width="465" height="558" alt="image" src="https://github.com/user-attachments/assets/062f851e-d927-4e95-a754-3f08e325c6eb" />
<img width="443" height="552" alt="image" src="https://github.com/user-attachments/assets/efa72d11-d8c5-439f-8f1f-ab009149b4c2" />
<img width="461" height="553" alt="image" src="https://github.com/user-attachments/assets/2891bc42-74e0-4226-83b3-d172df920ad9" />


#### SNAT






   


# Basic Firewall Network Topology (WORK IN PROGRESS)

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
<img width="1602" height="1080" alt="image" src="https://github.com/user-attachments/assets/7104d22f-0ee7-49ab-af5c-990cb63285da" />


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
int e0/1
	no shutdown
	description To-Admin_out
	ip address 11.11.11.2 255.255.255.252
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
			set prefix-length 30
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
---

## Firewall Policies
I’ve configured the network to allow communication between the internal networks, provide Internet access, and expose the required services within the DMZ.

The FortiGate firewall policies control how this traffic is allowed between the internal networks, the Internet, and the DMZ. Policies are processed from top to bottom.

<img width="1331" height="340" alt="image" src="https://github.com/user-attachments/assets/d76b6ca9-50d2-4070-bd76-c5c98ca40861" />


### Policy 1: Internal-Network

This policy allows traffic between the two internal LANs through
port2 and port3.

<img width="997" height="967" alt="image" src="https://github.com/user-attachments/assets/6806b9a9-70ed-4915-af34-5770fe7111ed" />

### Policy 2: Internal Network to DMZ

This policy allows devices on the internal network to connect to the DMZ servers when needed. Only connections initiated by the internal network are allowed, while the DMZ servers cannot initiate connections back to the internal network.

<img width="996" height="963" alt="image" src="https://github.com/user-attachments/assets/b16b7e1f-defa-4b88-aea0-c924edae8264" />


### Policy 3: Internal-To-Outside

This policy allows the internal networks to access the Internet through the WAN interface. NAT is enabled for outbound traffic.

<img width="1016" height="929" alt="image" src="https://github.com/user-attachments/assets/624dd57b-e938-4fa7-b5ae-7ad93fc1e2c5" />

---
### Virtual IP 
The VIPs handle the DNAT for the DMZ servers. Each one forwards a specific public port on the FortiGate to the private IP and SSH port of the corresponding server, allowing external access while keeping the servers' private IP addresses hidden.

<img width="1217" height="157" alt="image" src="https://github.com/user-attachments/assets/0022e976-ec3e-42f7-8e90-bb25c334424c" />

---


### Policy 4: Outside-To-SRVR1

This policy allows external SSH access to SRVR1 through public port
`2221`. The DNAT configuration forwards the connection to SRVR1 on
port `22`.

<img width="995" height="955" alt="image" src="https://github.com/user-attachments/assets/ec41316c-12f4-4092-8a0e-dc452a1de9a3" />

#### DNAT Configuration: 
<img width="465" height="558" alt="image" src="https://github.com/user-attachments/assets/a4767ad6-f239-475d-ab4d-0dfe2e52bd04" />


### Policy 5: Outside-To-SRVR2

This policy allows external SSH access to SRVR2 through public port
`2222`. The DNAT configuration forwards the connection to SRVR2 on
port `22`.

<img width="994" height="954" alt="image" src="https://github.com/user-attachments/assets/5403419d-7bad-4aa6-902a-75d48cda940e" />


#### DNAT Configuration: 
<img width="443" height="552" alt="image" src="https://github.com/user-attachments/assets/d95dc702-adda-468d-9de6-8a74cb916232" />


### Policy 6: Outside-To-SRVR3

This policy allows external SSH access to SRVR3 through public port
`2223`. The DNAT configuration forwards the connection to SRVR3 on
port `22`.

<img width="997" height="953" alt="image" src="https://github.com/user-attachments/assets/79c017de-87ee-42d2-9c30-283fe466deb9" />

#### DNAT Configuration: 
<img width="461" height="553" alt="image" src="https://github.com/user-attachments/assets/8f50410a-6d78-44ee-9b51-de0b918b046f" />


### Policy 7: Implicit Deny

The implicit deny rule blocks traffic that does not match any of the
explicit firewall policies above.

--- 


## DMZ Servers Configuration

For the DMZ servers, I used routers to represent the servers in
the lab environment. This keeps the setup simple while still allowing basic
connectivity and remote-access testing, such as ping and SSH.

Each simulated server was assigned a static IP address in the DMZ and a
default route pointing to the FortiGate.

| Server | IP Address | Default Gateway |
|:---:|:---:|:---:|
| SRVR1 | 172.16.100.5/28 | 172.16.100.1 |
| SRVR2 | 172.16.100.6/28 | 172.16.100.1 |
| SRVR3 | 172.16.100.7/28 | 172.16.100.1 |

SSH was configured on the routers to simulate remote access to the servers.
SSH version 2 was enabled with a local admin account and RSA keys, while the
VTY lines were configured to accept SSH connections.

| Server | username | secret (login) | secret (privileged EXEC) |
|:---:|:---:|:---:|:---:|
| SRVR1 | admin | password123 | password123 |
| SRVR2 | admin | password123 | password123 |
| SRVR3 | admin | password123 | password123 |

### Configuration

#### Server 1: 
```cisco
!!!! INITIAL CONFIG
enable
config terminal
hostname SRVR1
int e0/0
	description To-Switch_DMZ
	no shutdown
	ip address 172.16.100.5 255.255.255.240
exit
ip route 0.0.0.0 0.0.0.0 172.16.100.1


!!!! SSH CONFIG
enable secret password123
username admin secret password123
ip domain-name company.dmz
crypto key generate rsa general-keys modulus 2048
ip ssh version 2
line vty 0 4
	transport input ssh
	login local
	logging synchronous
end
```

#### Server 2: 
```cisco
!!!! INITIAL CONFIG
enable
config terminal
hostname SRVR2
int e0/0
	description To-Switch_DMZ
	no shutdown
	ip address 172.16.100.6 255.255.255.240
exit
ip route 0.0.0.0 0.0.0.0 172.16.100.1


!!!! SSH CONFIG
enable secret password123
username admin secret password123
ip domain-name company.dmz
crypto key generate rsa general-keys modulus 2048
ip ssh version 2
line vty 0 4
	transport input ssh
	login local
	logging synchronous
end
```

#### Server 3: 
```cisco
!!!! INITIAL CONFIG
enable
config terminal
hostname SRVR3
int e0/0
	description To-Switch_DMZ
	no shutdown
	ip address 172.16.100.7 255.255.255.240
exit
ip route 0.0.0.0 0.0.0.0 172.16.100.1


!!!! SSH CONFIG
enable secret password123
username admin secret password123
ip domain-name company.dmz
crypto key generate rsa general-keys modulus 2048
ip ssh version 2
line vty 0 4
	transport input ssh
	login local
	logging synchronous
end
```

---

## Administrator Access

Two administrators were used to test access to the DMZ. The internal
administrator connects to the DMZ servers from within the network, while
the external administrator connects through the Internet.

For lab purposes, I also used lightweight routers to simulate the administrator PCs.

<img width="1646" height="1080" alt="image" src="https://github.com/user-attachments/assets/e3ff6751-a69c-4305-b2fe-1f4248a61ee7" />

### Admin_in
```cisco
enable
configure terminal
hostname Admin_in
int e0/0
	description To_Switch1
	no shutdown
	ip address dhcp
end
```
### Admin_out
```cisco
enable
configure terminal
hostname Admin_out
int e0/0
	description To_INTERNET
	no shutdown
	ip address 11.11.11.1 255.255.255.252
end
```

--- 

## SSH Test
SSH connectivity was tested from both an internal network administrator and an external administrator to verify that the firewall policies and DNAT were working as expected. The internal administrator connected directly to the DMZ servers, while the external administrator accessed them through the configured public ports.

### Test 1: Internal-Network to Servers (SSH)
```cisco
ssh -l admin 172.16.100.5
```
<img width="762" height="518" alt="image" src="https://github.com/user-attachments/assets/a2e3feda-0004-4613-b378-fcca58d5c11c" />

&nbsp;

```cisco
ssh -l admin 172.16.100.6
```
<img width="764" height="518" alt="image" src="https://github.com/user-attachments/assets/8fa754e3-e3ac-4e93-a629-791ddec3d3c6" />

&nbsp;

```cisco
ssh -l admin 172.16.100.7
```
<img width="763" height="517" alt="image" src="https://github.com/user-attachments/assets/8ce42175-ded5-4870-bf27-6813acd856c5" />

&nbsp;

### Test 2 : Outside to Servers (SSH)

```cisco
ssh -p 2221 -l admin 100.1.1.1
```
<img width="762" height="518" alt="image" src="https://github.com/user-attachments/assets/b951eafe-e275-4c1d-ae03-a842eb4dab1c" />


&nbsp;

```cisco
ssh -p 2222 -l admin 100.1.1.1
```
<img width="762" height="522" alt="image" src="https://github.com/user-attachments/assets/970548a7-a7a9-4ae3-bb26-96a492c07697" />


&nbsp;

```cisco
ssh -p 2223 -l admin 100.1.1.1
```
<img width="760" height="513" alt="image" src="https://github.com/user-attachments/assets/78cca295-5846-4fda-a023-e6b5bc37bddb" />

---

## Observations

---

## Conclusion





   


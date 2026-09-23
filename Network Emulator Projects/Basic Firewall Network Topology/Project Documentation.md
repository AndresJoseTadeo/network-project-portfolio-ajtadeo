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

## Initial Firewall Configuration

I'm using FortiGate's `port1` as the management interface, connected to the emulator's Cloud0 interface, which provides connectivity to the VMware virtual network. This allows me to access the FortiGate GUI from my PC for initial configuration.

<img width="711" height="445" alt="image" src="https://github.com/user-attachments/assets/1a46c42c-9386-436f-8fba-a1b533199b54" />

### Configuration : 
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

### FortiGate's login page
<img width="1920" height="1036" alt="image" src="https://github.com/user-attachments/assets/021f49d1-b113-4091-80e5-4219c0e30c9c" />

### FortiGate's Dashboard
<img width="1920" height="1039" alt="image" src="https://github.com/user-attachments/assets/a3ae628d-d39b-4584-a3d3-cced28f02fbe" />

<br>

This web-based platform makes it easier to create, edit, and manage firewall security rules, internet access controls, and routing policies through a visual interface. 

<br>
It makes network configuration and monitoring more straightforward since you don't have to memorize a lot of command-line syntax, while also helping reduce configuration errors.


## Initial Device Configuration (Hostname, IP Addressing, Routing)
This section covers the initial device setup, including hostnames, IP addressing, and routing configuration. You can view the configuration details by expanding the summary section

<details>

<summary>Internal Network</summary>

### LAN 1:

### Switch 1
```cisco
enable
config terminal
hostname Switch1
```

### Router1
```cisco
enable
config terminal
hostname Router1
int e0/0
	no shutdown
	ip address 192.168.10.1 255.255.255.0
	ip helper-address 10.10.10.2
int e0/1
	no shutdown 
	ip address 10.10.10.1 255.255.255.252
	ip ospf 1 area 0
exit
router ospf 1
	router-id 10.255.255.1
	network 192.168.10.0 0.0.0.255 area 0
```

### LAN 2:

### Switch 2
```cisco
enable
config terminal
hostname Switch2
```

### Router2
```cisco
enable
config terminal
hostname Router2
int e0/0
	no shutdown
	ip address 192.168.20.1 255.255.255.0
	ip helper-address 10.10.20.2
int e0/1
	no shutdown 
	ip address 10.10.20.1 255.255.255.252
	ip ospf 1 area 0
exit
router ospf 1 
	router-id 10.255.255.2
	network 192.168.20.0 0.0.0.255 area 0
```
</details>

<details>


<summary>DMZ Network</summary>

### Switch_DMZ
```cisco
enable
config terminal
hostname Switch_DMZ
```

### Router1
```cisco
Router_DMZ
enable
config terminal
hostname Router_DMZ
int e0/0
	no shutdown
	ip address 172.16.100.1 255.255.255.240
int e0/1
	no shutdown 
	ip address 10.10.100.1 255.255.255.252
	ip ospf 1 area 0
exit
router ospf 1 
	router-id 10.255.255.4
	network 172.16.100.0 0.0.0.15 area 0
```
</details>



<details>
	
<summary>FortiGate Firewall</summary>

&nbsp;
```cisco
Note: I configured these settings through the FortiGate GUI dashboard.
I obtained the CLI commands shown below afterward by using the show commands to document my configuration.
```

### Global Config:
```
config system global
    set admintimeout 30
    set alias "FortiGate-VM64-KVM"
    set hostname "FortiGate"
    set timezone 04
end
```

### Interfaces:

```cisco
config system interface
    edit "port1"
        set vdom "root"
        set ip 192.168.222.141 255.255.255.0
        set allowaccess ping https ssh http
        set type physical
        set snmp-index 1
    next
    edit "port2"
        set vdom "root"
        set ip 10.10.10.2 255.255.255.252
        set allowaccess ping 
        set type physical
        set alias "TO-R1"
        set device-identification enable
        set lldp-transmission enable
        set role lan
        set snmp-index 2
    next
    edit "port3"
        set vdom "root"
        set ip 10.10.20.2 255.255.255.252
        set allowaccess ping 
        set type physical
        set alias "TO-R2"
        set device-identification enable
        set lldp-transmission enable
        set role lan
        set snmp-index 3
    next
    edit "port4"
        set vdom "root"
        set ip 10.10.100.2 255.255.255.252
        set type physical
        set alias "TO-DMZ"
        set role dmz
        set snmp-index 4
    next
    edit "port5"
        set vdom "root"
        set ip 100.1.1.1 255.255.255.252
        set allowaccess ping 
        set type physical
        set alias "WAN-PORT"
        set lldp-reception enable
        set role wan
        set snmp-index 9
        config ipv6
            set ip6-send-adv enable
            set ip6-other-flag enable
        end
```

### OSPF (Internal Network - DMZ Communication)
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
    config redistribute "rip"
    end
    config redistribute "bgp"
    end
    config redistribute "isis"
    end
end
```
### Static Routing (To WAN)
```cisco
config router static
    edit 1
        set gateway 100.1.1.2
        set device "port5"
    next
end

```
</details>

<details>
<summary>External Network</summary>
	
### INTERNET
```cisco
enable
conf terminal
hostname INTERNET
int e0/0
	no shutdown
	ip address 100.1.1.2 255.255.255.252
int loopback 0
	no shutdown
	ip address 8.8.8.8 255.255.255.255
do wr
```

</details>


## Project Requirements

### 1. Communication Between LAN 1 and LAN 2

The first requirement is to allow communication between the two internal LANs.

<img width="972" height="949" alt="image" src="https://github.com/user-attachments/assets/6fe4b782-4cf9-4410-b3ae-0717b8a5db7b" />

&nbsp;

I configured the network so that devices in LAN 1 can communicate with devices in LAN 2 and vice versa.

<img width="2000" height="677" alt="image" src="https://github.com/user-attachments/assets/40a3c44c-59c4-4fa0-b4e8-62f472d8f531" />

&nbsp;

<img width="1920" height="920" alt="image" src="https://github.com/user-attachments/assets/815cbb63-1527-4d15-a501-11812f9984b0" />


### 2. DHCP for the Internal Networks

* Brief description

Firewall interface configuration
<details>
	config system dhcp server
    edit 1
        set ntp-service local
        set default-gateway 10.255.1.1
        set netmask 255.255.255.0
        set interface "fortilink"
        config ip-range
            edit 1
                set start-ip 10.255.1.2
                set end-ip 10.255.1.254
            next
        end
        set vci-match enable
        set vci-string "FortiSwitch" "FortiExtender"
    next
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
    next
</details>
    
  * DHCP relay agent configuration

### 3. DMZ Network Segmentation

* Brief description
* **Configuration**

  * Firewall configuration

### 4. Internal Network to DMZ

* Brief description
* **Configuration**

### 5. Source NAT (SNAT)

* Brief description
* **Configuration**

### 6. Destination NAT (DNAT) and Virtual IPs

* Brief description
* **Configuration**

## Testing

* Test results and verification

## Observations

* Key observations

## Conclusion

* Brief summary of the completed configuration and results

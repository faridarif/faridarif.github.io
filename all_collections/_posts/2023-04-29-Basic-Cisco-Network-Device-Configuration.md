---
layout: post
title: Basic Cisco Network Device Configuration
date: 2023-04-29
categories: ["Networking"]
---

## Switch Configuration
1) Enter privilege mode :
```bash
Switch> enable
```
OR
```bash
Switch> en
```

2) Enter global configuration mode :
```bash
Switch# configure terminal
```
OR
```bash
Switch# config term
```

3) Disables the switch from translating unfamiliar words (typos) into IP addresses :
```bash
Switch(config)# no ip domain-lookup
```

4) Specifies the name for the switch :
```bash
Switch(config)# hostname <switch's name>
```

5) Set a password that is required for any user to enter global configuration mode :
```bash
Switch(config)# enable secret <password>
```

6) Set a password for the console port of the switch and enables password verification at the terminal login session :
- *Line con 0 refers to **the console port of the switch** (it is usually either an RJ-45 or, on newer devices, a USB port). The console port is used to physically connect (a PC or laptop) to the device.*
```bash
Switch(config)# line con 0
Switch(config-line)# password <password>
Switch(config-line)# login
```

7) Set a password for VTY (Virtual Terminal) and enables password verification at the virtual terminal login session :
- *The virtual terminal or “VTY” lines are virtual lines that allow connecting to the device using telnet or Secure Shell (SSH). Cisco devices can have up to 16 VTY lines. You can determine how many VTY lines you have by issuing “line vty 0 ?” from global configuration mode. The value 0 4 in this command means "from virtual terminal 0 to virtual terminal 4" because in this situation we assume that this switch can have only 5 user login to virtual terminal (telnet or SSH) at the same time. The device can allow 5 simultaneous virtual connections. By default five vty lines (0–4) are open.*
```bash
Switch(config)# line vty 0 4
Switch(config-line)# password <password>
Switch(config-line)# login
```

8) Encrypt the password :
- *This command **obscures all clear-text passwords in the configuration using a Vigenere cipher**.*
```bash
Switch(config)# service password-encryption
```

9) Set a login banner, known as the message of the day (MOTD) banner :
```bash
Switch(config)# banner motd #
```

10) Save the configuration :
- *This command save the running configuration to the startup file on non-volatile random access memory (NVRAM).*
```bash
Switch(config)# copy running-config startup-config
```
OR
```bash
Switch(config)# wr
```

11) List all the IP interfaces or physical port and VLAN in the switch :
- *This command show the type (Fast Ethernet, Gigabit Ethernet, VLAN), IP address, status and protocol of the interface*
```bash
Switch# show ip interface brief
```
OR
```bash
Switch# sh ip int brief
```

12) Enter the interface configuration mode for the specific VLAN ID, assign an IP address to that VLAN and brings up the interface :
- *In Switch, an IP address will **only** be assigned to Switch Virtual Interface (VLAN interface) when we want to use Layer 3 routing (IP routing) such as interVLAN routing and management VLAN*
- *[Switch Virtual Interface](https://en.wikipedia.org/wiki/Switch_virtual_interface) or SVI is a virtual routed interface or logical routed interface that represents all of the port belonging to that VLAN*
- *IP address is assign to VLAN interface. IP address can't be assign to interface port. By default, all interface port is in VLAN 1 (Native VLAN by default)*
- *There is differences between switch port and routed port. Routed port is a physical port that can route IP traffic (Layer 3/Network Layer) to another device.*
- *This command shows the example of creating VLAN interaface using the "int vlan" command and assigning an IP address to the interface VLAN 1*
- *"no shutdown" command will brings the interface "up", while the "shutdown" command will shuts down the interface and the status of the interface will be "administratively down" *
```bash
Switch(config)# int vlan 1
Switch(config-if)# ip address <ip-address> <subnet-mask>
Switch(config-if)# no shutdown
```

13) Configure the IP default gateway for the switch :
- *Default gateway is an address of the interface port (Router) that is connecting directly to the switch*
- *Default gateway must be configured on a layer 2 switch to allow traffic to be sent to remote network or remote subnet (different network or different subnet)*
- *If default gateway is not configured, any traffic that receive on switch would not be able to be sent again to source address (Sender would not get any reply from the switch)*
- *Layer 2 switch is different from layer 3 switch (multilayer switch), layer 3 switch has an IP routing feature that allow traffic to be sent to other network (layer 3 IP routing)*
- *In layer 3 switch (Multilayer Switch), we don't need default gateway, we just need to enable IP routing feature*
- **If the Layer 2 switch is directly connected to Multilayer Switch, the default  gateway is only be using when IP routing feature on Multilayer Switch is disabled. Otherwise, we need to use IP default route on Layer 2 switch when IP routing feature is enabled. [Reference](https://www.udemy.com/course/complete-networking-fundamentals-course-ccna-start/learn/lecture/8297266#overview)**
```bash
Switch(config)# ip default-gateway 192.168.1.1
```

14) Enabling port security on the switch :
- *Port Security limits the number of valid MAC addresses allowed to transmit data through a switch port (If an unknown MAC address sends data through the port, it will dropped by the switch and take an action based on Violation Modes - Default Mode: Shutdown)*
- *Before enable the port security, the port must be configure as static port whether access or trunk*
```bash
Switch(config)# int FastEthernet 0/1
Switch(config)# switchport mode access
Switch(config)# switchport port-security
```


## Router Configuration
1) Enter privilege mode :
```bash
Router> enable
```
OR
```bash
Router> en
```

2) Enter global configuration mode :
```bash
Router# configure terminal
```
OR
```bash
Router# config term
```

3) Disables the router from translating unfamiliar words (typos) into IP addresses :
```bash
Router(config)# no ip domain-lookup
```

4) Specifies the name for the router :
```bash
Router(config)# hostname <switch's name>
```

5) Set a password that is required for any user to enter global configuration mode :
```bash
Router(config)# enable secret <password>
```

6) Set a password for the console port of the router and enables password verification at the terminal login session :
- *Line con 0 refers to **the console port of the router** (it is usually either an RJ-45 or, on newer devices, a USB port). The console port is used to physically connect (a PC or laptop) to the device.*
```bash
Router(config)# line con 0
Router(config-line)# password <password>
Router(config-line)# login
```

7) Set a password for VTY (Virtual Terminal) and enables password verification at the virtual terminal login session :
- *The virtual terminal or “VTY” lines are virtual lines that allow connecting to the device using telnet or Secure Shell (SSH). Cisco devices can have up to 16 VTY lines. You can determine how many VTY lines you have by issuing “line vty 0 ?” from global configuration mode. The value 0 4 in this command means "from virtual terminal 0 to virtual terminal 4" because in this situation we assume that this switch can have only 5 user login to virtual terminal (telnet or SSH) at the same time. The device can allow 5 simultaneous virtual connections. By default five vty lines (0–4) are open.*
```bash
Router(config)# line vty 0 4
Router(config-line)# password <password>
Router(config-line)# login
```

8) Encrypt the password :
- *This command **obscures all clear-text passwords in the configuration using a Vigenere cipher**.*
```bash
Router(config)# service password-encryption
```

9) Set a login banner, known as the message of the day (MOTD) banner :
```bash
Router(config)# banner motd #
```

10) Save the configuration :
- *This command save the running configuration to the startup file on non-volatile random access memory (NVRAM).*
```bash
Router(config)# copy running-config startup-config
```
OR
```bash
Router(config)# wr
```

11) List all the IP interfaces or physical port of the router :
- *This command show the type (Fast Ethernet, Gigabit Ethernet, Subinterface, VLAN), IP address, status and protocol of the interface*
```bash
Router# show ip interface brief
```
OR
```bash
Router# sh ip int brief
```

12) Enter the interface configuration mode for the specific interface port and assign an IP address to that interface port :
- *This command shows the example of assigning an IP address to the interface FastEthernet 0/1*
- *"no shutdown" command will brings the interface "up", while the "shutdown" command will shuts down the interface and the status of the interface will be "administratively down"*
```bash
Router(config)# int FastEthernet 0/1
Router(config-if)# ip address <ip-address> <subnet-mask>
Router(config-if)# no shutdown
```


## Reference :
- https://www.netwrix.com/cisco_commands_cheat_sheet.html
- https://www.cisco.com/c/en/us/td/docs/routers/access/800M/software/800MSCG/routconf.html

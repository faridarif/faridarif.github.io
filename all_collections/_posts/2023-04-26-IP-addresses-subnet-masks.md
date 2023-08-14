---
layout: post
title: IP Addresses & Subnet Masks
date: 2023-04-26
categories: ["Networking"]
---
**Note: IP addresses and subnet masks are mathematical and technical topics. In order to build a better understanding of IP addresses and subnet masks, we must first understand the basic operation of computers and the concepts of binary and hexadecimal. We must understand the term '*bit*' in computer operation. In binary representation, each bit can be either 0 or 1, which is essential when dealing with IP addresses and subnet masks because each part of the IP address can only be one of these two values. Understanding binary and hexadecimal allows us to grasp how IP addresses are represented in these formats, making it easier to work with them, especially when subnetting or dealing with IPv6 addresses that are expressed in hexadecimal. Do your own studies and refer to all the references included in the References section for more information.**

## IP Addresses

An IP address, short for Internet Protocol address, is a unique numerical label assigned to each device connected to a computer network that uses the Internet Protocol for communication. IP addresses are essential for **identifying and locating** devices in a network. They come in two versions which are IPv4 and IPv6.

### IPv4 (Internet Protocol version 4)

This is the most commonly used version of the IP address. An IPv4 address is a **32-bit** number represented as **four decimal numbers** (each ranging from 0 to 255), **separated by periods (dots)**. For example, an IPv4 address looks like this : 
```
192.168.1.1
```

Each decimal number is called an **octet** because it represents **8 bits** of the 32-bit IPv4 address. In binary, each octet can have a value between 0 (00000000 in binary) and 255 (11111111 in binary).

To convert an IPv4 address to binary, we convert each octet to its 8-bit binary representation and concatenate them. For example, let's convert `192.168.1.1` to binary :
```
IP Address            :  192   .  168  .   1    .   1
Binary Representation : 11000000.1010000.00000001.00000001
```

In IPv4, IP address classes (classful IP addressing) were used to categorize IP addresses into different classes based on the range of values in their first octet, which is the first 8 bits (from the left) of the IP address. These classes helped define the default subnet masks and the number of hosts per network for each class. IP address classes were primarily used in the early days of the internet before the adoption of Classless Inter-Domain Routing (CIDR) in the 1990s. **This classful IP addressing is no longer used nowadays. However, it is worth knowing and understanding this classful address mask, especially when configuring network devices.** \*Read the CIDR section for more details.

 - **Class A** :
	- First bit (from the left) always set to 0 : `01111111 = 127` & `00000000 = 0`
	- Range of the **first octet** (in decimal) : 0 to 127 (from `0.0.0.0`  to  `127.0.0.0`). So `10.0.0.0` is a Class A address, which is a common example of a Class A address.
	- Default subnet mask : 255.0.0.0 ( /8 in CIDR notation ).
	- Large number of networks (2^7 = 128) and a large number of hosts per network (2^24 -2 = 16,777,214).
	- Support 16,777,214 host addresses.

- **Class B** :
	- First two bits (from the left) always set to 10 : `10000000 = 128` & `10111111 = 191`
	- Range of the **first octet** (in decimal) : 128 to 191 (from `128.0.0.0`  to  `191.0.0.0`). So `172.16.0.0` is a Class B address, which is a common example of a Class B address.
	- Default subnet mask : 255.255.0.0 ( /16 in CIDR notation ).
	- Moderate number of networks (2^13 = 16,384) and a moderate number of hosts per network (2^16 -2 = 65,534).
	- Support 65,534 host addresses.

- **Class C** :
	- First three bits (from the left) always set to 110 : `11000000 = 192` & `11011111 = 223`
	- Range of the **first octet** (in decimal) : 192 to 223 (from `192.0.0.0`  to  `223.0.0.0`). So `192.168.11.0` is a Class C address, which is a common example of a Class C address.
	- Default subnet mask : 255.255.255.0 ( /24 in CIDR notation ).
	- Large number of networks (2^21 = 2,097,152) but a small number of hosts per network (2^8 - 2 =254).
	- Support 254 host addresses.

- **Class D** :
	- First four bits (from the left) always set to 1110 : `11100000 = 224` & `11101111 = 239`
	- Range of the **first octet** (in decimal) : 224 to 239 (from `224.0.0.0`  to  `239.0.0.0`).
	- Class D addresses are reserved for multicast groups and not used for traditional unicast communication.

- **Class E** :
	- First four bits (from the left) always set to 1111 : `11110000 = 240` & `11111111 = 255`
	- Range of the **first octet** (in decimal) : 240 to 255 (from `240.0.0.0`  to  `255.0.0.0`).
	- Class E addresses were reserved for experimental purposes and not used for regular communication.

Why do we subtract 2 when calculating the numbers of hosts per network above (for example: 2^8 - 2 = 254)? This subtraction is done to account for the network address and the broadcast address, which cannot be assigned to individual hosts. What is network address and broadcast address? \*Read the **Subnet Masks** section and the **Network Portion** section for more details.
- **Network Address** : The network address is the first address in a subnet (all host bits are set to 0 in the network address) and represents the subnet itself. For example, in a subnet with a subnet mask of `255.255.255.0`  ( /24 ), the network address would be the IP address with all the host bits set to 0 (such as `192.168.1.0`). For example :
```
IP Address            : 192.168.1.0
Subnet Mask           : 255.255.255.0
Subnet Mask in Binary : 11111111.11111111.11111111.00000000 = All host bits set to 0
```
- **Broadcast Address** : The broadcast address is the last address in a subnet and is used to send a message to all hosts within that subnet. All host bits are set to 1 in the broadcast address. Using the same example subnet from Network Address above, the broadcast address would be `192.168.1.255`. For example :
```
IP Address            : 192.168.1.0
Subnet Mask           : 255.255.255.0
Subnet Mask in Binary : 11111111.11111111.11111111.11111111 = All host bits set to 1
```
Since these two addresses are reserved and cannot be assigned to individual devices, they reduce the number of usable host addresses in a subnet. **That's why we subtract 2 from the total number of possible addresses to calculate the number of hosts per network**.

### IPv6 (Internet Protocol version 6)

An IPv6 address is a **128-bit** number represented in **hexadecimal format**. It consists of **eight groups of four hexadecimal digits separated by colons**. For example, an IPv6 address looks like this : 
```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```
Leading zeros within each group can be **omitted**, and consecutive groups of zeros can be **replaced with a double colon (::)**. However, the double colon can only be used **once in an IPv6 address**. For example :
```
2001:0db8:85a3:0000:0000:8a2e:0370:7334 = 2001:0db8:85a3:0:0:8a2e:0370:7334

2001:0000:0000:0000:0000:0000:0000:0001 = 2001::1
```

IPv6 defines several address types for different purposes :

- **Unicast Address** : Used to identify a single network interface (or a device). There are three types of unicast addresses :
	- Global Unicast Address (Routable on the **public internet**).
	- Link-Local Address (Used for communication on a **local network segment**, which similar to private addresses in IPv4).
	- Unique Local Address (Similar to private IPv4 addresses, but globally unique within an organization or site).

- **Multicast Address** : Used for **one-to-many** communication. Packets sent to a multicast address are delivered to **multiple hosts** that have joined the corresponding multicast group (not all hosts in the network).
   
- **Anycast Address** : Assigned to multiple interfaces, a packet sent to an anycast address is **delivered to the nearest** (in terms of routing distance) of those interfaces.

In IPv6, the subnetting process is much simpler compared to IPv4 because IPv6 allocates a large number of addresses to each subnet. Subnetting is done by assigning a prefix length to the IPv6 address. The prefix length indicates the number of bits set to 1 in the subnet prefix (because IPv6 is a 128-bit number). For example, a prefix length of **/64** means the **first 64 bits (from the left) are the network portion**, and the **remaining 64 bits are hosts**.

IPv6 has built-in mechanisms for stateless address autoconfiguration (SLAAC), where hosts on a network can **automatically generate** their IPv6 addresses **without needing a DHCP server**. This is accomplished using the network's prefix and a unique identifier (for example, MAC address) to form the host portion of the IPv6 address. IPv6 provides an incredibly vast number of unique addresses, ensuring the scalability of the internet for the foreseeable future.

## Subnet Masks

A subnet, short for Subnetwork, is a division of a larger network into smaller, more manageable segments. **The subnet address is a unique identifier for a specific subnetwork within a larger network**. Subnetting allows network administrators to improve network performance, security, and efficiency by grouping devices with similar network requirements together.

A subnet mask is a 32-bit value (in IPv4) used to identify the network and host portions of an IP address. In an IPv4 address, the subnet mask is expressed in decimal format (such as `255.255.255.0`), and aligns with the number of bits borrowed for the subnet. The subnet mask contains a series of contiguous 1s followed by contiguous 0s. The 1s represent the network portion, and the 0s represent the host portion. For example, let's consider a subnet mask of `255.255.255.0` ( /24 ) :
```
11111111.11111111.11111111.00000000
```
In this subnet mask, the first 24 bits (from the left) are set to 1, which means the first 24 bits of the IP address are part of the network portion, and the last 8 bits are set to 0, representing the host portion.

### Subnet Address Calculation

To calculate the subnet address for a given IP address and subnet mask, perform a bitwise AND operation between the IP address and the subnet mask (The result will be the subnet address). For example, consider the IP address `192.168.1.50` and the subnet mask `255.255.255.224` ( /27 ) :
```
IP Address     : 11000000.10101000.00000001.00110010 = 192.168.1.50
Subnet Mask    : 11111111.11111111.11111111.11100000 = 255.255.255.224
Bitwise AND    : 11000000.10101000.00000001.00100000 = 192.168.1.32

Subnet Address :   192   .   168  .   1    .   32
```
So, the subnet address for the IP address `192.168.1.50` with a /27 subnet mask (27 bits of 1 in the subnet mask above) is `192.168.1.32`.

## Network vs Host Portion in IPv4 Addresses

### Network Portion

The network portion is the part of the address that identifies the specific network to which the device belongs. It helps routers and switches determine how to route [packets](https://faridarif.github.io/posts/packets-vs-frames/) between different networks. All devices within the **same network** share the **same network portion of their IP addresses**. The length of the network portion is determined by the subnet mask applied to the IP address. The 1s represent the network portion, and the 0s represent the host portion. For example, let's consider an IP address `192.168.1.1` with a subnet mask of `255.255.255.0` ( /24 ). The subnet mask in binary is :
```
IP Address           : 192.168.1.1
IP Address in Binary : 11000000.10101000.00000001.00000001 = 192.168.1.1
Subnet Mask          : 11111111.11111111.11111111.00000000 = 255.255.255.0
Bitwise AND          : 11000000.10101000.00000001.00000000 = 192.168.1.0

Network Address      : 192.168.1.0
```
In this case, the **first three octets (24 bits)** are the network portion, and the **last octet (8 bits)** is the host portion. So, the network portion or network address of this IP address is `192.168.1.0`. **The network address is the address that identifies an entire network**. **In terms of words, "*network address*" is often used interchangeably with "*network portion*" when discussing IP addressing**. If, for example, an IP address of `192.168.1.231` with a subnet mask of `255.255.255.224` ( /27 ) :
```
IP Address           : 192.168.1.231
IP Address in Binary : 11000000.10101000.00000001.11100111 = 192.168.1.231
Subnet Mask          : 11111111.11111111.11111111.11100000 = 255.255.255.224
Bitwise AND          : 11000000.10101000.00000001.11100000 = 192.168.1.224

Network Address      : 192.168.1.224
```
So, the network portion or network address of this IP address is `192.168.1.224`. Is subnet address same as network address? The terms "**subnet address**" and "**network address**" are **not interchangeable**, as a network address refers to the overall address of a network, while a subnet address refers to a specific subnetwork within that larger network.

### Host Portion

The host portion is the part of the address that identifies a specific device within a network. Each device within a network must have a unique host portion to ensure proper communication. For example the subnet of `255.255.255.0` ( /24 ) in binary :
```
11111111.11111111.11111111.00000000
```
**The network portion is on the left hand side (1s), and the host portion is on the right hand side (0s)**.

To calculate the number of hosts in a subnet, count the number of host bits in the subnet mask (the number of bits set to 0), and use the formula : **2^n - 2**, where n is the number of host bits. For example :
```
Subnet Mask           : 255.255.255.0
Subnet Mask in Binary : 11111111.11111111.11111111.00000000

Formula : 2^8 - 2 = 254 hosts    
```
Which means there are 254 specific addresses available to be assigned to a device within this network. Another example :
```
Subnet Mask           : 255.255.255.224
Subnet Mask in Binary : 11111111.11111111.11111111.11100000

Formula : 2^5 - 2 = 30 hosts  
```

**The subtraction of 2 accounts for the network address and the broadcast address**, which are not usable by individual hosts. Remember that the network address has **all host bits set to 0**. For example of subnet mask `255.255.255.0` : 
```
Network address: 11111111.11111111.11111111.00000000
```
While the broadcast address has **all host bits set to 1** :
```
Broadcast address: 11111111.11111111.11111111.11111111
```

**The host address is the specific address assigned to a device within a network that is within the range defined by the host portion of the IP address**. **The terms "*host portion*" and "*host address*" are also used interchangeably when discussing IP addressing**. Using the example of IP address `192.168.1.1`, the range of host addresses within this network is **from `192.168.1.1` to `192.168.1.254`**. One specific device can have an IP address of `192.168.1.3`, and another specific device can have an IP address of `192.168.1.251` within this network.

## CIDR Notation

CIDR, which stands for Classless Inter-Domain Routing, is a method for representing and allocating IP addresses in a more flexible and efficient manner compared to the older classful addressing system. CIDR replaces classful IP addressing.

### Classful Addressing vs CIDR (Classless Addressing)

- **Classful Addressing** : In the early days of the internet, IP addresses were assigned based on classful addressing, which divided the IP address space into fixed classes (A, B, C, D, and E). Each class had a **predetermined** number of bits for the **network portion** and the **host portion**. This fixed allocation led to significant wastage of IP address space, especially in cases where organizations were assigned larger address blocks that they needed.
- **CIDR (Classless Addressing)** : CIDR, on the other hand, does away with the strict class-based divisions and allows for **variable-length subnet masks** (VLSM). With CIDR, an IP address is represented with both IP address and the subnet mask together, using the notation "/*prefix_length*". VLSM uses the notation `192.168.1.0/24` rather than the notation `192.168.1.0  255.255.255.0`. For example, `192.168.1.0/24` indicates that the **first 24 bits are the network portion**, and the **remaining 8 bits are for hosts**.

For example, an organization might need a subnet with only 30 hosts. Instead of allocating a whole Class C network with 254 usable addresses, CIDR allows for a /27 subnet. providing just the 30 addresses required. **No longer do we have class A, B, and C networks, where class A is always /8, class B is always /16, and class C is always /24**.

## References

- [Udemy - The Complete Networking Fundamentals Course](https://www.udemy.com/course/complete-networking-fundamentals-course-ccna-start/)
- [IP Characteristics and IPv4 Address Format](https://www.udemy.com/course/complete-networking-fundamentals-course-ccna-start/learn/lecture/7996258#content)
- [Network vs Hots Portion - The Complete Networking Fundamentals Course(Udemy)](https://www.udemy.com/course/complete-networking-fundamentals-course-ccna-start/learn/lecture/4482068#content)
- [Class A and B Addresses - The Complete Networking Fundamentals Course(Udemy)](https://www.udemy.com/course/complete-networking-fundamentals-course-ccna-start/learn/lecture/4482076#content)
- [Class C, D, and E Addresses - The Complete Networking Fundamentals Course(Udemy)](https://www.udemy.com/course/complete-networking-fundamentals-course-ccna-start/learn/lecture/7996268#content)
- [Subnet Mask - The Complete Networking Fundamentals Course(Udemy)](https://www.udemy.com/course/complete-networking-fundamentals-course-ccna-start/learn/lecture/7996284#content)
- [Subnet Mask Cheat Sheet](https://www.freecodecamp.org/news/subnet-cheat-sheet-24-subnet-mask-30-26-27-29-and-other-ip-address-cidr-network-references/)
- [CIDR Part 1 & 2 - The Complete Networking Fundamentals Course(Udemy)](https://www.udemy.com/course/complete-networking-fundamentals-course-ccna-start/learn/lecture/7996296#overview)
- [CIDR Part 3 - The Complete Networking Fundamentals Course(Udemy)](https://www.udemy.com/course/complete-networking-fundamentals-course-ccna-start/learn/lecture/5579672#overview)
- [Binary? How does that works?](https://www.youtube.com/watch?v=Toa5-1i4wRE&list=PLhfrWIlLOoKMCYDh94esrjiB5nUSPqMh0&index=35)
- [Binary IP Conversions](https://www.youtube.com/watch?v=TjW7yCqsHr8&list=PLhfrWIlLOoKMCYDh94esrjiB5nUSPqMh0&index=36)
- [Hexadecimal to Decimal](https://www.youtube.com/watch?v=bqF0zoGTaY0&list=PLhfrWIlLOoKMCYDh94esrjiB5nUSPqMh0&index=38)
- [IPv6 address](https://en.wikipedia.org/wiki/IPv6_address)
- [Wildcard mask](https://en.wikipedia.org/wiki/Wildcard_mask)

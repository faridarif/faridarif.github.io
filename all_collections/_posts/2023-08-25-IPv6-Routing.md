---
layout: post
title: IPv6 Routing Configurations (Static & Dynamic Routing)
date: 2023-08-25
categories: ["Networking"]
---
**Note: [Fundamental Routing Concepts - Routing (Concepts, Protocols and Configurations)](https://faridarif.github.io/posts/routing/), [EIGRP (Enhanced Interior Gateway Routing Protocol)](https://faridarif.github.io/posts/EIGRP/) and [OSPF (Open Shortest Path First)](https://faridarif.github.io/posts/OSPF/)**

## Enable IPv6 Routing on Router

```bash
Router(config)# ipv6 unicast-routing
```

## IPv6 Static Routing

- **ipv6-prefix** : Destination network address of the remote network to be added to the routing table.
- **prefix-length** : Prefix length of the remote network to be added to the routing table.
- **ipv6-address** : The next-hop router's IPv6 address.
- **exit-interface** : The outgoing interface (one of the router's own interfaces that is directly connected to the next-hop router) to forward packets to the destination network.

```bash
Router-1(config)# ipv6 route <ipv6-prefix>/prefix-length [<ipv6-address> or <exit-interface>]
```

**Recursive Static Route :**

```bash
Router-1(config)# ipv6 route 2001:DB8:ACAD:2::/64 2001:DB8:ACAD:4::2
```

**Directly Attached Static Route :**

```bash
Router-1(config)# ipv6 route 2001:DB8:ACAD:2::/64 Serial0/0/0
```

**Fully Specified Static Route :**

```bash
Router-1(config)# ipv6 route 2001:DB8:ACAD:2::/64 Serial0/0/0 2001:DB8:ACAD:4::2
```

**Default Static IPv6 Route :**

- **::/0** : Matches any IPv6 prefix regardless of prefix length.
- **ipv6-address** : The next-hop router's IPv6 address.
- **exit-interface** : The outgoing interface (one of the router's own interfaces that is directly connected to the next-hop router) to forward packets to the destination network.

```bash
Router-1(config)# ipv6 route ::/0 [<ipv6-address> or <exit-interface>]
```

**IPv6 Floating Static Route :**

```bash
Router-1(config)# ipv6 route ::/0 2001:DB8:ACAD:4::2 5
```
### Verify IPv6 Static Routes

This command shows the IPv6 routing table :

```bash
Router# show ipv6 route static
```

## EIGRP for IPv6

1) Initialize the EIGRP routing process and enter the EIGRP router configuration mode :

- AS number functions as a process ID. The AS number used for EIGRP configuration is only significant to the EIGRP routing domain.
- AS number configured in a router must be the same as the neighbor's AS number.

```bash
Router(config)# ipv6 router eigrp <AS-number>
```

2) Set the EIGRP Router-ID for a router :

- Router-ID is used to uniquely identify each router in the EIGRP routing domain.
- Router-ID is in IP address format, which is four decimal numbers separated by dots. For example, 2.2.2.2

```bash
Router(config-router)# eigrp router-id 2.2.2.2
```

3) By default, the EIGRP for IPv6 process is in a **shutdown** state and the command below is required to activate the EIGRP for IPv6 process :

```bash
Router(config-router)# no shutdown
```

4) Unlike EIGRP for IPv4 which uses the `network` command, EIGRP for IPv6 is configured directly on the interface using the `ipv6 eigrp <AS-number>` interface configuration command :

```bash
Router(config)# int GigabitEthernet 0/0
Router(config-if)# ipv6 eigrp 2
```

5) Suppress routing updates on an interface :

- Passive interfaces prevent EIGRP routing updates over a specified router interface.

```bash
Router(config-router)# passive-interface <interface-id>
```

6) EIGRP for IPv6 (Default Route Propagation)

```bash
Router(config-router)# redistribute static
```

7) EIGRP for IPv6 (Hello and Hold Timers) :

```bash
Router(config-if)# ipv6 hello-interval eigrp <AS-number> <seconds>
Router(config-if)# ipv6 hold-time eigrp <AS-number> <seconds>
```
## OSPFv3

1) Initialize OSPF routing process and enter the OSPF router configuration mode :

- The process ID value represents a number between 1 and 65,535 and is selected by the network administrator. The process ID value is locally significant.
- It is considered best practice to use the same process ID on all OSPF routers.

```bash
Router(config)# ipv6 router ospf <process-id>
```

2) Explicitly configure Router-ID for a router :

- Router-ID is in IP address format, which is four decimal numbers separated by dots. For example, 1.1.2.2

```bash
Router(config-router)# router-id 1.1.2.2
```

3) Unlike OSPFv2 which uses the `network` command, OSPFv3 is configured directly on the interface using the `ipv6 ospf <process-id> area <area-id>` interface configuration command :

```bash
Router(config)# int GigabitEthernet 0/0
Router(config-if)# ipv6 ospf 10 area 1
```

4) Suppress OSPFv3 routing on an interface :

- Passive interfaces prevent the transmission of routing messages through an interface, but still allow that network to be advertised to other routers.

```bash
Router(config-router)# passive-interface <interface-id>
```

5) OSPFv3 Tuning (Default Route Propagation) :

- This command instructs the router to be the source of the default route information and propagate the default static route in OSPF updates.

```
Router(config-router)# default-information originate
```

6) OSPFv3 Tuning (Modify OSPF Intervals) :

- To restore the default interval, use the "no" form in this command.

```
Router(config-if)# ipv6 ospf hello-interval <seconds>
Router(config-if)# ipv6 ospf dead-interval <seconds>
```

## References

- [Chapter 1: Routing Concepts - Cisco NetAcad Powerpoint Presentation Slide](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/miscellaneous/Chapter%201%20Routing%20Concepts.pptx)
- [Chapter 2: Static Routing - Cisco NetAcad Powerpoint Presentation Slide](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/miscellaneous/Chapter%202%20Static%20Routing.pptx)
- [Understand and Use the Enhanced Interior Gateway Routing Protocol - Cisco Docs](https://www.cisco.com/c/en/us/support/docs/ip/enhanced-interior-gateway-routing-protocol-eigrp/16406-eigrp-toc.html)
- [EIGRP - Cisco Article](https://www.ciscopress.com/articles/article.asp?p=2999383&seqNum=2)
- [Module 6: EIGRP - Cisco NetAcad Powerpoint Presentation Slide](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/miscellaneous/Module%206%20EIGRP.pptx)
- [Module 7: EIGRP Tuning and Troubleshooting - Cisco NetAcad Powerpoint Presentation Slide](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/miscellaneous/Module%207%20EIGRP%20Tuning%20and%20Troubleshooting.pptx)
- [Community Question - The Cisco Learning Network](https://learningnetwork.cisco.com/s/question/0D53i00000Kt5O1CAJ/eigrp-properties-as-distance-vector-link-state)
- [Understand Open Shortest Path First (OSPF) - Cisco Docs](https://www.cisco.com/c/en/us/support/docs/ip/open-shortest-path-first-ospf/7039-1.html)
- [IP Routing: OSPF Configuration Guide - Cisco Docs](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_ospf/configuration/xe-16/iro-xe-16-book/iro-cfg.html)
- [Cisco IOS IP Routing: OSPF Command Reference](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_ospf/command/iro-cr-book/m_ospf-a1.html#)

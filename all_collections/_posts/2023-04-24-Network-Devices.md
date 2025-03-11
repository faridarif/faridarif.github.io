---
layout: post
title: Network Devices (Repeater, Hub, Switch, and Router)
date: 2023-04-24
categories: ["Networking"]
---

**Note: There are several other networking devices and equipment commonly used in various network environments. However, Repeaters, Hubs, Switches, and Routers are the most basic and crucial networking devices used in various network environments. These devices form the building blocks of networking infrastructure, and their proper deployment and configuration are essential for efficient data transmission, secure communication, and reliable network connectivity. This blog post will not explain in detail how these devices work.**

---
## **Repeater**

Repeater is a basic networking device used to extend the range of a network by amplifying or regenerating signals. The repeater would repeat the signal from one port to another. The repeater basically didn't understand the actual signal, but it just amplified the signal from one port to another. Its primary function is to receive weak signals, clean them up, and then retransmit them at a higher power level to cover longer distances. Repeater works at the **Physical Layer (Layer 1)** of the OSI model. Repeater only helps to extend network reach. It does not increase the overall network bandwidth. Repeater typically have two ports, which are an input port and an output port. **Repeaters** operate in **full-duplex** mode, because they receive and transmit at the same time. However, **the repeater user** is usually operating in **half-duplex** mode.

- In traditional wired networks, a repeater is used to overcome the limitations of signal attenuation over long distances. As data travels along a network cable, it gradually loses strength due to resistance and other factors that may lead to data loss and poor network performance.

- In modern networking, the functionality of repeaters is often integrated into other devices such as routers and access points.

- In wireless networks, repeaters play a critical role in extending the coverage area. When a wireless signal weakens due to distance or obstacles like walls and interference, a repeater placed in the vicinity can receive the signal, amplify it, and then retransmit it to reach areas that were previously out of range.

---
## **Hub**

Hub is often referred to as a "*multi-port repeater*" because it shares similarities with a repeater and has multiple ports to connect multiple devices. The primary function of a hub is to receive incoming data packets from one connected device and **broadcast** them, or repeat the signal to all other connected devices. This broadcasting approach is often referred to as "*flooding*". Hubs also didn't understand the actual signal. This means that Hubs do not have the intelligence to determine which specific device should receive that data packet or signal. Both repeater and hub work at the **Physical Layer (Layer 1)** of the OSI model. **Hubs** operates in **half-duplex** mode. Only one device can send or receive data at a time.

- The Wi-Fi network is essentially a "*hub in the air*". While both hubs and Wi-Fi networks use broadcast communication, the main difference lies in the medium and the underlying technologies. Hubs are devices used in wired networks, and Wi-Fi is a technology used in wireless networks.

---
## **Switch**

Switch is a fundamental networking device used to connect multiple devices **within a local network** and facilitate communication between them. It operates at the **Data Link Layer (Layer 2)** of the OSI model. The big difference between a switch and a hub is that a switch has intelligence. A switch actually reads the [frames](https://faridarif.github.io/posts/packets-vs-frames/) received on [Ethernet](https://faridarif.github.io/posts/Ethernet/), which are sent by the connected device. Switches use **MAC address table** to determine which specific device should receive those frames. Unlike hubs, which use broadcast communication and transmit data to all connected devices, a switch uses a process called "*packet switching*" to intelligently forward data packets to their intended destination. Key characteristics and functionalities of a switch include :

- **MAC Address Learning** : Switches maintain a table called the MAC address table or forwarding table. When a data packet arrives at a switch's port, it reads the source MAC address from the packet and records it in the MAC address table, associating it with the port from which the packet was received.
- **Forwarding and Filtering** : When a data packet arrives at a switch's port and needs to be forwarded to a specific destination, the switch consults its MAC address table to determine which port the destination MAC address is associated with. It forwards the packet only to the port where the destination device is located, rather than broadcasting it to all ports, like a hub.
- **Switching Paths** : A switch can establish multiple simultaneous communication paths between pairs of devices connected to it. This allows for simultaneous communication between different pairs of devices within the same network, without causing collisions, as each pair of devices has its own dedicated communication path.
- **Full-Duplex Communication** : Switches support full-duplex communication, which means that data can be sent and received simultaneously on a single connection.

Switches have largely replaced hubs in modern networks due to their intelligent and efficient data forwarding capabilities. By selectively forwarding data only to the relevant ports, switches significantly reduce network congestion, collisions, and unnecessary traffic.

---
## **Router**

Router is a critical networking device that operates at the **Network Layer (Layer 3)** of the OSI model. It is used to **connect different networks** together and facilitate the exchange of data between them. Routers play a fundamental role in directing data packets to their intended destinations, enabling communication between devices on **different networks**. Key characteristics and functionalities of a router include :

- **Routing** : Routers use routing tables and algorithms to determine the best path for data packets to reach their destinations. They examine the destination IP address of each incoming data packet and compare it to their routing table to determine the next hop or the next router on the path to the destination network.
- **Interconnect Networks** : Routers are used to interconnect various networks, such as local area networks (LANs) and wide area networks (WANs). For example, a home router connects the devices within the home network to the internet, which is a global network.
- **IP Addressing** : Routers rely on IP addresses to identify and route data packets. They perform IP address translation (NAT - Network Address Translation) to allow multiple devices in a private network to share a single public IP address for internet access.
- **Dynamic Routing Protocols** : Routers use dynamic routing protocols (for example, OSPF, RIP, EIGRP, BGP) to exchange routing information with other routers, enabling them to automatically update their routing tables in response to network changes.
- **Traffic Segmentation** : Routers can segment network traffic into different subnets, creating separate broadcast domains and logically isolating devices within those subnets.

Routers are crucial for the proper functioning of the internet and many other complex networks. The data packets are directed efficiently through the network, ensuring that information reaches its intended destination while taking the most optimal path possible.

---
## **References**

- [Repeater](https://en.wikipedia.org/wiki/Repeater)
- [Ethernet hub](https://en.wikipedia.org/wiki/Ethernet_hub)
- [Network switch](https://en.wikipedia.org/wiki/Network_switch)
- [Router (computing)](https://en.wikipedia.org/wiki/Router_(computing))
- [Simplex, Duplex, Offset and Split](https://www.hamradioschool.com/post/simplex-duplex-offset-and-split)
- [YouTube - Network Devices Part 1](https://www.youtube.com/watch?v=CdoG4tWNPqs&list=PLhfrWIlLOoKMCYDh94esrjiB5nUSPqMh0&index=26)
- [YouTube - Network Devices Part 2](https://www.youtube.com/watch?v=gdV0T91Bvt4&list=PLhfrWIlLOoKMCYDh94esrjiB5nUSPqMh0&index=27)
- [Udemy - The Complete Networking Fundamentals Course](https://www.udemy.com/course/complete-networking-fundamentals-course-ccna-start/)
- [What is a network switch? Switch vs. router](https://www.cloudflare.com/learning/network-layer/what-is-a-network-switch/)

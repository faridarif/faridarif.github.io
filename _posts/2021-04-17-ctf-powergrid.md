---
title: "PowerGrid (Vulnhub)"
date: 2021-04-17
layout: single
permalink: /ctf-writeups/powergrid-vh/
tags: [Hacking, CTF]
---

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid.png){:.align-center}

## Overview

Kita diberikan pengenalan seperti berikut di halaman Vulnhub <https://www.vulnhub.com/entry/powergrid-101,485/> :


Description

Cyber criminals have taken over the energy grid across Europe. As a member of the security service, you’re tasked with breaking into their server, gaining root access, and preventing them from launching their malware before it’s too late.

We know from previous intelligence that this group sometimes use weak passwords. We recommend you look at this attack vector first – make sure you configure your tools properly. We do not have time to waste.

Unfortunately, the criminals have started a 3 hour clock. Can you get to their server in time before their malware is deployed and they destroy the evidence on their server?

This exercise is designed to be completed in one sitting. Shutting down the virtual machine will not pause the timer. **After the timer has finished, the CTF machine will be shut down and you will be unable to boot it. Please keep a local backup of the CTF prior to starting, in case you wish to attempt a second time**.

If you are to succeed, I strongly recommend reading these points:

-   Keep a local backup before starting in case you run out of time
-   You will need a basic understanding of the GPG tool and how it works
-   Configure your tools so they work at the maximum/hardest level possible. Make sure you are looping around the correct thing, if you know what I mean
-   Getting the initial shell is possibly the longest part.
-   There are four flags in total. Each flag file will guide you to the next area
-   This virtual machine has been tested in VirtualBox only. I cannot guarantee it will work on VMWare, but it should be okay.

## Reconnaissance

### /etc/hosts

Kadang kala mengingati *IP address* bagi sesebuah *machine* itu amatlah sukar bagi sesetengah orang.Oleh itu, kita terjemahkan *IP address* itu kepada *'Local DNS'* dengan memasukkan *IP address* itu ke dalam fail `/etc/hosts`.Dengan kaedah sebegini, kita hanya perlu mengingati *domain names* yang kita berikan kepada *IP address* tersebut berbanding mengingati keseluruhan *IP address* itu.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-etc-hosts.png){:.align-center}

### Nmap Scan

Hanya 3 *ports* yang *open* iaitu 80 (http), 143 (imap) dan 993 (ssl/imap) dan juga 1 *filtered port* iaitu 514 (shell)

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-nmap.png){:.align-center}

## Enumeration

Halaman utama website :

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-main-page.png){:.align-center}

Kita hanya diberikan masa 180 minit atau 3 jam untuk 'root' *machine* ini.Walau bagaimanapun, terdapat 3 *potential user* iaitu `deez1`, `p48` dan `all2`.Pada 'description' di atas, ada dinyatakan bahawa "this group sometimes use weak passwords. We recommend you look at this attack vector first" yang bermaksud berkemungkinan kita boleh menggunakan kaedah *brute forcing*.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-gobuster.png){:.align-center}

Terdapat satu *web directory* iaitu `zmail` yang meminta kita untuk *login*.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-zmail-page.png){:.align-center}

Kita boleh melakukan *brute forcing* pada *login page* ini.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-brute-force.png){:.align-center}

Seterusnya kita boleh log masuk dan kemudian terdapat satu lagi *login page* yang berlainan.Dengan menggunakan *username* dan *password* yang sama, kita berjaya log masuk ke laman tersebut.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-zmalil-page2.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-zmail-page2-login.png){:.align-center}

Terdapat satu *email* yang mengandungi *'encrypted message'* dalam format PGP.Kita memerlukan *'private key'* dan juga *password* untuk membacanya.Kita dapat lihat bahawa pengguna *p48* menggunakan *password* yang sama bagi kedua-dua *login page*,berkemungkinan kita boleh menggunakan *password* yang sama bagi membaca mesej tersebut.Namun kita perlukan *'private key'*.Oleh itu, kita akan tinggalkannya buat sementara waktu kerana kita tiada *'private key'* dan teruskan proses 'enumeration'.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-zmail-page-version.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-searchsploit.png){:.align-center}

Terdapat 'public exploit' bagi *Roundcube version 1.2.2* dan ianya adalah Remote Code Execution.

## Exploiting

**Exploit**: **<https://www.exploit-db.com/exploits/40892>**

### Proof of Concept

Melihat pada Proof of Concept pada exploit itu, kita akan lakukan PHP Code Injection yang mana kita akan 'inject' PHP code pada parameter '_subject'.

**Normal request :**

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-poc1.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-poc2.png){:.align-center}

Kemudian kita akan ubah suai pada parameter *'_from'* dan *'_subject'*.Pada parameter *'_from'*, kita akan ubahnya kepada tempat atau *directory* untuk kita letakkan 'malicious PHP file' dalam sistem itu.Manakala pada parameter *'_subject'* pula, kita akan 'inject' PHP code.Jadi pada parameter *'_subject'* akan mengandungi 'malicious PHP code'

**Malicious request :**

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-poc3.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-poc4.png){:.align-center}

### Execute Arbitrary Commands

**Payload :** `<?php echo passthru($_GET[‘cmd’]); ?>`

Pastikan anda *URL-encode payload* itu

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-exploit1.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-exploit2.png){:.align-center}

### Reverse Shell

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-exploit3.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-revershell.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-flag1.png){:.align-center}

Seperti yang kita temui sebelum ini, pengguna *p48* menggunakan *password* yang sama.Oleh itu, kita boleh `su` kepada *p48* dengan menggunakan *password* yang sama.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-su-p48.png){:.align-center}

Dalam `/home/p48`, terdapat *'private key'* yang kita boleh gunakan untuk 'decrypt' message sebelum ini.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-p48-1.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-decrypt-pgp-message.png){:.align-center}

Ternyata ianya adalah *SSH private key*.Machine yang kita berada sekarang tiada *ssh service*.Pada `flag1.txt`, terdapat petunjuk iaitu *'pivoting'*.Jadi kita akan lakukan *pivoting*.

### Pivoting

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-ip-addr.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-ssh-success.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-flag2.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-gtfobins.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-flag3.png){:.align-center}

## Privilege Escalation

Petunjuk yang seterusnya pada `flag3.txt` iaitu 'pivoting backwards'.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/powergrid-flag4.png){:.align-center}

### HAPPY HACKING !

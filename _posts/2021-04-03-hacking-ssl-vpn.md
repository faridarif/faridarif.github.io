---
title: "Hacking ZALORA SSL VPN - Part 1: System Analysis, Attack Vectors and Finding Bugs Pulse Secure SSL VPN"
date: 2021-04-03
layout: single
permalink: /infosec/hacking-zalora-ssl-vpn-part-1-system-analysis-attack-vectors-and-finding-bugs-pulse-secure-ssl-vpn/
tags: [Hacking , Infomation Security ]
---
## Pengenalan
SSL VPN adalah salah satu daripada *Virtual Private Network* yang bertujuan melindungi aset korporat daripada terdedah kepada internet.SSL VPN juga adalah satu-satunya jalan untuk masuk ke dalam intranet (rangkaian tertutup) sesebuah oraganisasi korporat melalui internet atau *remote* (kawalan jarak jauh).Namun,bagaimana jika SSL VPN tersebut mempunyai kelemahan yang kritikal yang membolehkan penggodam untuk masuk ke dalam intranet sesebuah organisasi korporat itu? Hal ini membolehkan penggodam untuk mengambil alih (take over) semua pengguna yang bersambung (connecting) dengan pelayan SSL VPN tersebut.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/ssl-vpn.png){: .align-center}

## Jailbreaking SSL VPN

SSL VPN bersifat *black box* dan *closed source*.Kebanyakan SSL VPN hanya menyediakan "Restricted Shell".Oleh itu, "jailbreak" adalah satu-satunya pilihan bagi para penyelidik (researchers) untuk menjadikan "black box" itu kepada "great box".

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/restricted-shell.png){: .align-center}

Namun bagaimana jika *disk* itu adalah *encrypted*?

![Linux Booting Process](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/linux-booting-process.png){: .align-center}

Kaedah yang biasa digunakan untuk memecahkan *encryption* adalah dengan menganalisis atau "reverse" vmlinuz kernel untuk mendapatkan kunci bagi *encryption* itu (encryption key).Walaubagaimanapun,kita akan fokus kepada "stash" yang lain iaitu */sbin/init* yang menggunakan kaedah "memory forensics".

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/step-booting-process.png){: .align-center}

KIta semua mengetahui bahawa *booting process* bermula dengan *bios*, *boot loader*, dan *kernel* yang kemudiannya melancarkan (initialize) operating system itu.Bagi Pulse Secure,ianya akan memaparkan "Press Enter to view or update your appliance settings." dan apabila kita menekan butang "Enter", ianya akan memaparkan satu lagi paparan yang membolehkan kita untuk ubah suai (modify) aturcaranya (settings).

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/next-booting-process.png){: .align-center}

Seterusnya, kita akan "suspend" *virtual machine* ini untuk seketika bagi mengimbas (scan) keseluruhan memori untuk lakukan *memory forensics* ke atasnya.Ketika melakukan *memory forensics*, kita dapat memerhatikan bahawa proses pelancaran itu adalah Perl script */home/bin/dsconfig.pl*

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/perl-script.png){: .align-center}

Kita boleh menggunkannya untuk "pop out" shell.KIta gantikan Perl script itu dengan */bin/sh* dan apabila kita menekan butang "Enter" sekali lagi, kita akan memperoleh *shell*.Dan sekarang kita telah memperoleh "full control of the system" dan seterusnya membolehkan kita untuk "debugging" dan menganalisis SSL VPN tersebut.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/patch-memory.png){: .align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/press-enter.png){: .align-center}

## Attack Vectors & Finding Bugs

Terdapat banyak *deamon* yang berjalan (running) dalam SSL VPN server tersebut dan ianya agak kompleks dan susah untuk dianalisis.Terdapat beberapa "feature" yang boleh dijadikan sebagai "attack vector" :

- WebVPN
- Native script language extensions
- Multi-layered architecture problems

### WebVPN

WebVPN adalah *proxy feature* yang mudah untuk digunakan kerana ianya mudah alih dan bersifat "clientless".Ianya boleh melakukan *proxy* terhadap kesemua trafik melalui Web Browser.WebVPN juga menyokong pelbagai jenis protokol seperti HTTP, FTP, TELNET, SSH dan sebagainya dan ianya juga mengendalikan pelbagai web resources seperti WebSocket, JavaScript dan Flash.Kita tidak perlu untuk memuat turun apa-apa perisisan, hanya dengan klik pada web browser, kita sudah boleh bersambung (connect) dengan pelbagai jenis *service*.Namun, ianya tidak mudah untuk membangunkan "feature" ini.Kebanyakan vendor memilih untuk membangunkannya sendiri.Pengendalian protokol dan web resources akan menyebabkan ianya terdedah kepada *memory bugs*.Tambahan pula, ianya memerlukan kesedaran keselamatan yang tinggi seperti melakukan *Debug Function*, *Logging Sensitive Data*, *Encryption* dan *Information Exposed Carelessly*.Dalam WebVPN implementation juga,mereka sering "copy" kod daripada open source project.Permasalahannya disini adalah apabila mereka "copy" kod, mereka juga sedang "copy" bugs.Jadi tidak mustahil untuk kita menjumpai bugs yang sama dalam beberapa WebVPN yang berlainan.Selain itu,ianya juga agak sukar untuk diselenggara dan dikemas kini.Oleh itu, tidak hairanlah WebVPN dikategorikan sebagai "attack vector" yang paling serius dibawah SSL VPN.

### Native script language extensions

Kebanyakan SSL VPN mempunyai native script language extensions mereka yang tersendiri.Sama seperti PHP extensions yang mengekalkan C atau Perl yang mengekalkan C++.Kita boleh menggunakan extensions ini bagi melakukan *encoding/decoding* untuk meningkatkan kecekapan yang menjadikannya "vulnerable".Ianya boleh membawa kepada "type confusion" diantara *languages* yang berlainan.Seperti yang kita semua tahu, *string operation* selalunya sukar bagi C language seperti *buffer size calculation*, *dangerous functions* dan *missunderstood functions*.

```c
ret = snprintf(buf, buf_size, format, ...) ;
left_buf_size = buf_size - ret;
```

Pengiraan seperti di atas boleh menyebabkan *integer overflow*.Type confusions:

```perl
my ($var) = @_;
EXTENSION::C_funtion($var);
```

Ianya kelihatan serupa tetapi adakah ianya Perl string atau C string? Adakah anda boleh kenal pasti string tersebut? Tiada siapa tahu, dan ini menyebabkan masalah yang serius dan inilah bugs yang perlu di pandang serius oleh pengendali SSL VPN.

### Multi-layered architecture problems

Disebabkan ianya mempunyai "different standards" bagi memproses perkara yang sama, ketidakkonsistenan itu boleh mengakibatkan *Failed patterns* bagi *reverse proxy* + *java web* dan *customized (C/C++) web server* + RESTful API backend

- Abuse Regular Expression greddy mode to bypass path check
      ^/pulic/images/.+/(front | background)_.+
- Dispatched to backend PHP engine and access privileged pages

```
https://sslvpn/public/images/x/front_x/../../../../some.php
```



Bersambung pada bahagian seterusnya (part 2)

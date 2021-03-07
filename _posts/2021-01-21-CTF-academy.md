---
title: "Academy (HackTheBox)"
date: 2021-02-27
layout: single
permalink: /ctf-writeups/academy-htb/
tags: [Hacking, CTF]
---

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-htb.png){: .align-center}

## Scanning & Enumeration

Seperti biasa,kita mulakan dengan *nmap scan* :

- *nmap -A -T4 10.10.10.215*

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/nmap-academy.png){: .align-center}

Hanya 2 *ports* yang *open*.Kita akan mulakan dengan *enumerate website* pada *port* 80 tetapi sebelum itu perhatikan pada "Did not follow redirect to http://academy.htb" yang bermaksud adanya URL redirection pada *web application* itu.URL redirection adalah suatu teknik untuk memberikan lebih daripada satu alamat URL kepada sesuatu laman web atau seluruh aplikasi laman web tersebut.Hal ini bermaksud jika kita menggunakan alamat URL http://10.10.10.215 dan http://academy.htb,kedua-dua alamat URL itu akan membawa kita ke dua laman web yang berbeza.Anda boleh mencubanya sendiri.Oleh itu,kita perlu edit */etc/hosts* kita dengan meletakkan ip address *machine* itu ke dalam */etc/hosts* seperti yang ditunjukkan dalam gambar di bawah.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-etc-hosts.png){: .align-center}

*Web page* :

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-webpage.png){: .align-center}

Perkara pertama kita perlu lakukan apabila menjumpai sesuatu laman web itu adalah *directory brute-forcing* :

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-gobuster.png){: .align-center}

Setelah *enumerate* seluruh *web directory*,keseluruhannya kelihatan seperti normal dan tiada sebarang *hint* yang boleh kita dapati.Oleh itu,hanya *login.php* , *admin.php* dan *register.php* sahaja yang boleh dianggap sebegai *potential vulnerabilties*.Disebabkan kita tidak menjumpai apa-apa *credentials* untuk kita *brute-forcing* *login page* tersebut,maka kita akan cuba untuk *register*.Setiap kali *enumerate website*,pastikan kita sentiasa *intercept* sebarang *http request* menggunakan *Burp Suite* untuk menganalisis *http request* tersebut.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-register-intercept.png){: .align-center}

Kita menjumpai *"roleid"* parameter yang kita boleh *"abuse"*.*Roleid* parameter digunakan untuk mengenalpasti kebolehaksesan sesuatu *"user"* atau pengguna itu samaada pengguna itu adalah *administrator* atau hanya pengguna biasa.Oleh itu,jika kita tukar nilai *"roleid"* itu,kita mungkin boleh menukar status kita daripada pengguna biasa atau *"user"* kepada *administrator*.*Administrator* kebiasaanya berada pada nilai 1

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-success-register.png){: .align-center}

Kita telah berjaya *register*.Seterusya kita cuba untuk login sebagai *"admin"* menggunakan *credential* yang kita telah register sebentar tadi di *web directory* /admin.php

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-admin-login-page.png){: .align-center}

Kita telah berjaya login sebagai *"admin"*

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-admin-page.png){: .align-center}

"Fix issue with dev-staging-01.academy.htb" adalah "pending" dan ianya menunjukkan terdapat subdomain iaitu "dev-staging-01.academy.htb".Oleh itu,tambah *dev-staging-01.academy.htb* kedalam */etc/hots* seperti yang ditunjukkan dalam gambar di bawah

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-etc-hosts.png){: .align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-dev-staging.png){: .align-center}

Ianya mendedahkan *sensitive informations* dan kita mendapati bahawa ianya menggunakan *PHP Laravel Framework*

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-sensative-information.png){: .align-center}

## Exploitation

*Searchsploit* menunjukkan terdapat *public exploit* untuk *PHP Laravel Framework* yang menggunakan *token Unserialize RCE* dan ianya adalah *metasploit module*

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-searchsploit.png){: .align-center}

*run msfconsole* dan *set APP_KEY , RHOSTS , VHOST and LHOST*

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-msfconsole.png){: .align-center}

Kita telah berjaya dapatkan *reverse shell*

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-session.png){: .align-center}

*Spawn tty shell* :

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-spawn-shell.png){: .align-center}

Setelah *enumerate* setiap directory dalam */var/www/html*,kita menjumpai *"potential password"* dalam */var/www/html/academy/.env*

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-potential-pass.png){: .align-center}

*/home directory* :

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-home-directory.png){: .align-center}

*User flag* berada dalam */home/cry0l1t3*,maka *cry0l1t3* adalah *"potential user"*

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-cry0l1t3.png){: .align-center}

Dengan *potential password* dan *potential user* yang kita temui sebentar tadi,kita boleh cuba untuk *login* dengan menggunakan *ssh*

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-ssh.png){: .align-center}

Kita berjaya *login* sebagai *cry0l1t3*

## PrivEsc

- cry0l1t3@academy:~$ id

Menggunakan *command* di atas kita dapat melihat *information* tentang *user* atau pengguna *cry0l1t3* dan kita mendapati bahawa *cry0l1t3* berada dalam *groups* "adm".Adm adalah salah satu daripada *"system groups"* yang berperanan sebagai tugas pemantauan sistem (system monitoring tasks).Semua ahli dalam *groups* "adm" boleh melihat atau membaca (read) dan memantau kebanyakan fail *log* yang berada dalam /var/log.Oleh itu,kita boleh membaca fail *log* yang berada /var/log/audit.Kita boleh memindahkan semua fail yang berada dalam /var/log/audit itu ke *local machine* bagi memudahkan kita untuk menganalisis fail-fail tersebut.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-wget-audit.png){: .align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-audit-grep.png){: .align-center}

Kita menjumpai *credential* yang di "encode" dalam format hex.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-hex.png){: .align-center}

Setelah di "decode" *strings* tersebut,ianya adalah *password* bagi pengguna "mrb3n".

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-su-mrb3n.png){: .align-center}

Disebabkan kita memiliki *password* bagi pengguna "mrb3n",kita boleh menggunakan "sudo".

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-sudo.png){: .align-center}

Seperti yang kita dapat lihat,kita boleh *"execute"* fail /usr/bin/composer menggunakan sudo.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-gtfobins.png){: .align-center}

Gtfobins adalah sumber terbaik untuk mencari *exploit* bagi melakukan PrivEsc.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/academy-root.png){: .align-center}

Tahniah ! Kita sudah berjaya *"PAWNED"* *machine* ini !

### HAPPY HACKING ! 

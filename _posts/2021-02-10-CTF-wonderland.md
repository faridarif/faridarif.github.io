---
title: "Wonderland (TryHackMe)"
date: 2021-02-10
layout: single
permalink: /ctf-writeups/wonderland-thm/
tags: [Hacking, CTF]
---

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/wonderland.png){:.align-center}

## Enumeration

Seperti biasa,kita mulakan dengan *nmap scan* :

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-nmap.png){:.align-center}

Hanya dua *ports* yang *open* iaitu *port* 22 (ssh) dan *port* 80 (http).

**Website** :

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-webpage.png){:.align-center}

Pada *source code webpage* itu,terdapat satu *jpeg* file yang didalamnya tersembunyi satu lagi file (hint.txt).

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-steghide.png){:.align-center}  

Didalam *hint.txt* itu,tertulis "follow the r a b b i t".Buat masa ini,kita masih tidak tahu apa yang dimaksudkan dengan "follow the r a b b i t" itu.

**Gobuster** :

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-gobuster-r.png){:.align-center}

Terdapat satu *web directory* "/r"

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-webpage-r.png){:.align-center}

Melihat pada *web directory* "/r" itu,tiada apa-apa yang menarik di situ.Disebabkan ia tertulis "Keep Going" pada webpage itu,apa kata kita teruskan *directory bruteforcing* pada directory "/r" itu.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-gobuster-a.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-webpage-a.png){:.align-center}

Sekarang ini barulah kita faham bahawa "follow the r a b b i t" itu bermaksud web directory iaitu /r/a/b/b/i/t/.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-webpage-rabbit.png){:.align-center}

Melihat pada *source webpage* itu,kita mendapati bahawa terdapat *username* dan juga *password*.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-webpage-rabbit-source.png){:.align-center}
 
## Exploiting 

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-ssh-alice.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-ls.png){:.align-center}

Terdapat "root.txt" file pada alice home directory "/home/alice" tetapi ianya milik "root" dan kita tiada *permission* ke atasnya.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-question-hint.png){:.align-center}
 
Setelah melihat pada "question hint" itu,kita boleh simpulkan bahawa mungkin "user.txt" berada dalam "root directory".
 
**User flag** :
 
![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-ls-root.png){:.align-center}
 
![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alcie-user-flag.png){:.align-center}
 
## PrivEsc
 
![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-sudo-l.png){:.align-center}
 
Melihat pada "sudo -l" menyatakan bahawa kita hanya boleh "run" *python3.6* dan *walrus_and_the_carpenter.py* sebagai "rabbit".

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-walrus.png){:.align-center}
 
Pada bahagian paling atas,kita dapat lihat bahawa satu python module "random".Disebabkan kita tidak boleh "write" atau ubahsuai *walrus_and_the_carpenter.py* itu,kita akan "hijack" python module itu dengan "create" satu file "random.py".

Random.py :

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-random-py.png){:.align-center}
 
![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-ls-random-py.png){:.align-center}
 
**Rabbit** :

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-sudo-command.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-rabbit-revshell.png){:.align-center}
 
Kita berjaya "escalate our privileges" kepada "rabbit" user !

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-teaparty.png){:.align-center}

Didalam */home/rabbit*,terdapat satu "binary file" iaitu "teaParty".

Kita cuba pindahkan "taeParty" itu ke *local machine* supaya mudah untuk kita analisis "binary file" itu.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-http-server.png){:.align-center}
 
![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-elf.png){:.align-center}
 
Ianya adalah **ELF 64-bit** yang menandakan bahawa ianya adalah "Linux Executable file".Kita boleh lakukan "strings command" untuk melihat kandungan "binary file" ini. 

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-strings.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-execute-teaparty.png){:.align-center}

**"Strings" command** menunjukkan bahawa terdapat "date command" yang di "execute" tanpa menetapkan sebarang "path".Oleh itu,kita boleh "abuse" "vulnerability" ini dengan mengeksport "our own $PATH".

**$PATH** :

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-nano-failed.png){:.align-center}

Kita menghadapi masalah untuk "run" *nano* sebagai "rabbit".Oleh itu,kita kembali kepada user "alice" dan "create" satu file "date" didalam */home/alice*.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-date.png){:.align-center}
  
**Hatter** :

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-hatter.png){:.align-center}

Great, sekarang kita adalah "Hatter" !

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-ls-hatter.png){:.align-center}

Kita menjumpai file "password.txt" dalam */home/hatter* yang mana ianya adalah *password* bagi user "hatter" kerana apabila kita "run" "sudo -l" dengan menggunakan password itu,ia menyatakan "Sorry, user hatter may not run sudo on wonderland." yang memberi maksud bahawa password yang kita masukkan itu adalah password yang betul.Jika password itu salah,maka ia sepatutnya akan menyatakan "Sorry, try again.".Oleh itu,kita boleh "login ssh" sebagai "hatter".

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-ssh-hatter.png){:.align-center}

Setelah saya melakukan beberapa "basic enumeration",saya mendapati bahawa "perl" mempunyai capability set: cap_setuid+ep

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-perl.png){:.align-center}

**Root** :

**gtfobins** adalah rujukan terbaik.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-gtfobins.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/alice-root.png){:.align-center}

Yeay, akhirnya kita sudah "pwned this machine" ! ROOT !

### HAPPY HACKING !


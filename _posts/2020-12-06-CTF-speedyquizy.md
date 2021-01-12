---
title: "Mobile: SpeedyQuizy.apk (Wargames)"
date: 2020-12-06
layout: single
permalink: /ctf-writeups/speedyquizy-wgmy/
tags: [Hacking, CTF]
---

### Category: Mobile

### Challenge Name: SpeedyQuizy

## Introduction

Kita diberikan *android mobile application file* (apk).Kita akan gunakan *jadx-gui* untuk menganalisis *code file* tersebut

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/speedyquizy.png){:.align-center}

Kita dapat lihat bahawa keseluruhan *code* itu adalah tentang *connection* antara *application* itu dengan server www2.wargames.my:8080.Oleh itu,kita akan gunakan netcat untuk *connect* dengan server itu

- nc www2.wargames.my 8080

Kita berjaya *establish connection* dengan server itu dan kita diminta untuk menjawab 3 soalan dalam masa 4 saat.Soalan adalah rawak dan merangkumi topik aritmetik,*cryptography* dan *networking*

## Scripting

*Python script to automated the process*:

```python
import socket
import codecs

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s: 
	socket.socket(("www2.wargames.my", 8080))
	
	print(repr(s.recv(1024)))
	
	print("ok")
	s.send("ok".encode())
	
	for _ in range(3):
		print(repr(s.recv(1024)))
		q = s.recv(1024)
		print(repr(q))
		
		q = q.decode().lower().replace("\n", "").replace(".", "").replace("?", "")
		
		nums = [int(s) for s in q.split() if s.isdigit()]
		ans = ""
		
		if len(nums) == 2:
			if "add" in q or "plus" in q:
				ans = nums[0] + nums[1]
			elif "subtract" in q or "minus" in q:
				ans = nums[0] - nums[1]
			elif "multiply" in q or "times" in q:
				ans = nums[0] * nums[1]
			elif "devide" in q;
				ans = round(nums[0] / nums[1])
		elif "biggest port" in q:
			ans = 65535
		elif "reverse" in q:
			ans = q.split("reverse of ")[1].split(" ")[0][::-1]
		elif "monoalphabetic" in q or "shifted by 13" in q:
			ans = codecs.encode(q.split(" ")[-1], "rot_13")
		elif "dns zone transfer" in q:
			ans = "TCP"
		elif "tty" in q:
			ans = "teletype"
			
		print(ans)
		s.send(str(ans).encode())
		
	print(repr(s.recv(1024)))
	print(repr(s.recv(1024)))
```

*Run the script* dan kita akan dapat flag

- wgmy{418b3ea849ff3b93def86cfbc90440c1}

### HAPPY HACKING

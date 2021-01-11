---
title: "Reverse Engineering: Babyrev (Wargames)"
date: 2020-12-07
layout: single
permalink: /ctf-writeups/babyrev-wgmy/
tags: [Hacking, CTF]
---

### Wargames MY 2020 write-up
### Category: Reverse Engineering
### Challenge Name: babyrev


## Introduction

Dalam *challenge* ini,kita diberikan satu *executable file* (ELF) yang mana *executable file* itu meminta kita untuk memasukkan *password* yang betul dan panjangnya *password* hendaklah *32 characters*

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/elf-file.png){:.align-center}

## Decompilig an ELF File

Di *main() function*,kita boleh lihat bagaimana *executable file* itu mengawal input dan format *flag* yang akan kita buru sebentar lagi

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/main-func.png){:.align-center}

Dengan melihat *puts(s)* dan *strncmp(s)*,kita sudah boleh jangka bahawa *executable file* ini di tulis dengan menggunakan *C language*.Jadi,ayuh kita *rename* beberapa *variables* supaya mudah difahami

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/rename-var.png){:.align-center}

- local_68 = flag_string

- local_38 = user_input

Jadi,kita akan focus kepada bagaimana *flag* di proses.Jika kita perhatikan dengan teliti,kita dapat simpulkan bahawa *flag* = *password* , kerana *user_input* mestilah sama dengan flag_string dan characters ke 27 hingga 31 dalam flag_strings ialah "15963".Okay,sekarang mari kita lihat *value* bagi SHUFFLE dan XOR

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/shuffle.png){:.align-center}

SHUFLLE (HEX) : 07 04 15 12 1d 13 1b 08 1f 16 0f 06 0a 19 18 11 01 03 02 17 0d 14 05 00 0c 1c 0b 1a 0e 1e 09 10

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/XOR.png){:.align-center}

XOR (HEX) : 56 06 06 01 09 52 06 03 51 04 57 07 52 07 50 06 06 06 07 54 57 56 02 55 06 01 52 53 54 0f 54 03

## Calculating

Contoh python *script* :

```python3
from z3 import *

shuffle_bytes = 0x070415121d131b081f160f060a191811010302170d1405000c1c0b1a0e1e0910

xor_bytes = 0x56060601095206035104570752075006060607545756025506015253540f5403


#password

input_32_bytes = [
	z3.BitVec(name=f"x{i}",bv=8) for i in range(0,32)
]


#shuffled input

shuffle_result = []

for a in range(0,32):
	x = (shuffle_bytes >> a*8) & 0xff
	shuffle_result.append(input_32_bytes[x])

shuffle_result = shuffle_result[::-1]


get_xored = Concat(shuffle_result) ^ xor_bytes


the_password = Concat(input_32_bytes)


character_from_flag_string = Concat(shuffle_result[27],shuffle_result[28],shuffle_result[29]) ^ 0x53540f


a_solver = Solver()


a_solver.add(get_xored==the_password)


a_solver.add(character_from_flag_string==0x313539)

#printing the result

status = a_solver.check()

if str(status) == "sat":
	a_model = a_solver.model()
	for x in range (0,32):
		try:
			print(chr(a_model[input_32_bytes[x]].as_long()),end="")
		except:
			pass
	print()
```

*Running script* itu dan kita akan dapat *password* dan *flag*

![enter image description here](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/images/flag.png){:.align-center}

### HAPPY HACKING 

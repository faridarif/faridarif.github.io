The instruction : I made an **Xtra ObscuRe Service** that can encrypt and decrypt any message! Its **Very Secure!** Connect with `nc skrctf.me 3013`

Plaintext : hello
Encrypt  : Oy4+Fzc=

`hello` convert to Hex : 68656c6c6f
`Oy4+Fzc=` decrypt from Base64, the convert to Hex : 3b2e3e1737

68656c6c6f (`hello`)   XOR   3b2e3e1737 (`Oy4+Fzc=`)   =  534b527b58 (`SKR{X`)(part of the key)

534b527b58 (`SKR{X`)   XOR   3b2e3e1737 (`Oy4+Fzc=`)  =  68656c6c6f (`hello`)


The hint : *I wonder why I encrypt "SKR{" the encrypted message (base64) is all A's?*

Plaintext : SKR{
Encrypt  : AAAAAA==

`SKR{` convert to Hex : 534b527b
`AAAAAA==` decrypt from Base64, the convert to Hex : 00 00 00 00

534b527b (`SKR{`)   XOR   00 00 00 00 (`AAAAAA==`)   =  534b527b (`SKR{`)

534b527b (`SKR{`)   XOR   534b527b (`SKR{`)   =  00 00 00 00 

**So why I encrypt "SKR{" the encrypted message (base64) is all A's? Because the word "SKR{" itself is the key.**


So we need to Encrypt a long enough word to get the key (which is the flag) :

Plaintext : helloeveryonemynameisfaridariffromunikl
Encrypt  : Oy4+FzcqJDpDCjAAVRkmLj5YACoGFFItWgoiABAWEkNfAwg9IjkX

Plaintext (convert to Hex)   ^   Encrypt (decrypt from Base64, the convert to Hex)  =  flag


The flag : SKR{XOR_1s_n0t_@_5eCur3_3nCrypt10n}


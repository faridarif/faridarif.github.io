---
layout: post
title: Reverse Engineering : Find Me (Cyber Security Challenge - iTREXC)
date: 2023-10-21
categories: ["CTF", "Reverse Engineering"]
---

**Note: This is my write-up for one of the reverse engineering challenges in Cyber Security Challenge competition organized by iTREXC (4th International Innovation, Technology & Research Exhibition and Conference). This "Find Me" challenge is in Reverse Engineering category. The difficulty of this "Find Me" challenge is Easy.**

For this "Find Me" challenge, we were given an APK file (android application package). We can use JADX-GUI and Android Studio to reverse and run this APK file. Opening the APK file in JADX-GUI tells us that there's a Java Method (also known as function) named "*encrypt*" that encrypt a string using AES and Base64. We can see that the IV ([Initialization Vector](https://en.wikipedia.org/wiki/Initialization_vector)) and Secret Key (encryption key) used for AES encryption are `encryptionIntVec` and `dryEncyptionKey`.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/find-me-encrypt.png){:.align-center}

The application asked for user input in string format and stored that string in the "*userAnswer*" variable. The application then called the "*encrypt*" method above to encrypt the user input and stored that encrypted user input in the "*EncUserAns*" variable. The encrypted user input will then be compared with a hardcoded value : `ABxl23A0HbfOIPvu3jHQ==` (if both are equal, we will get the encoded string `CPmcjAylgzOLbc4sILX8UaAYf3C4UHYtFBzAcJ/LBWI=`; otherwise, the application will display the "Try again " message).

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/find-me-input-and-flag.png){:.align-center}

Decrypting `ABxl23A0HbfOIPvu3jHQ==` :

- **Key** : The secret key (encryption key) in UTF8 format.
- **IV** : The initialization vector in UTF8 format.
- **Mode** : CBC ([Cipher Block Chaining](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation))
- **Input** : Raw format
- **Output** : Raw format

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/find-me-input-aes-decrypt.png){:.align-center}

After decrypting the hardcoded value above, we know that the user input should be `c47m34G41n`. We can demonstrate this by running the application in Android Studio.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/find-me-proof.png){:.align-center}

By using the same method to decrypt the hardcoded encrypted user input above, we can decrypt the encoded string :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/find-me-flag-aes-decrypt.png){:.align-center}

After decrypting the encoded text, we get the flag : `uisctf{aes_is_easy}`.

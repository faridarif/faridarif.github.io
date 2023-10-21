---
layout: post
title: "Binary Exploitation : AdminShell (CyberSecurity Challenge - iTREXC 2023)"
date: 2023-10-21
categories: ["CTF", "Binary Exploitation"]
---
**Note: This is my write-up for one of the reverse engineering challenges in Cyber Security Challenge competition organized by iTREXC (4th International Innovation, Technology & Research Exhibition and Conference). This "AdminShell" challenge is in Binary Exploitation category. The difficulty of this "AdminShell" challenge is Easy/Medium.**

For this "AdminShell" challenge, we were given a Directory file that contains a "Dockerfile", an executable file named "main", and a fake flag file named "flag". The Dockerfile was redacted :

- A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble and image. [Docker](https://en.wikipedia.org/wiki/Docker_(software)) builds images automatically by reading the instructions from a Dockerfile. 

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/adminshell-dockerfile.png){:.align-center}

In the `bin` directory, there is a `main` executable file. We can execute the `main` file to know what the program does :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/adminshell-test-main.png){:.align-center}

The program asked the user to enter their name and password. Let's reverse engineer this `main` file with Ghidra. By looking at the `main` function, it seems that the program use `srand()` and `rand()` functions to initialize and generate random numbers. It generate random numbers every time we execute the program because the `seed` is set to `time null` (when we execute the program for the second time the random number will be different from the previous one). The program will then call the `validation()` function through a variable. The value in that variable will then be compared with a random number stored in another variable; if both are equal, the program will display the "Access granted!" message and call the `admin_shell()` function. Otherwise, the message "Wrong password, please try again later..." will be displayed.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/adminshell-ghidra-main.png){:.align-center}

The `validation()` function appears to be vulnerable to [Buffer Overflow](https://en.wikipedia.org/wiki/Buffer_overflow) attack :

- The password field accepts input more than its allocated size.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/adminshell-ghidra-validation.png){:.align-center}

The `checksec` tool tells us that there is no PIE in the executable file. It means that it is possible for us to do a buffer overflow attack on this executable file.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/adminshell-checksec.png){:.align-center}

The `admin_shell()` funtion :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/adminshell-ghidra-shell.png){:.align-center}

As a summary, we can overflow the password field to reach the `admin_shell()` function. The `admin_shell()` function will spawn us a shell. We can demonstrate this :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/adminshell-flag.png){:.align-center}

We successfully received the fake flag. To obtain the real flag, we need to exploit the real program file in the organizer server `admin.itrexc2023.capturextheflag.io`  on port `9999`.

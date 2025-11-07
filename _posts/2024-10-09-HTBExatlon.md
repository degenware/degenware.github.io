---
title: HackTheBox Exatlon writeup
date: 2024-10-09 08:00:00 +/-0000
categories:
  - Writeups
tags:
  - writeup
  - reverse-engineering
  - ctf
---
## Challenge Description

*Can you find the password?*

The file provided is an `elf` file

![](/assets/img/all/5TTLS0m7.jpg)

## Solution

The program asks for a password

![](/assets/img/all/xMKxzZ5m.jpg)

Running `strings` on the file returned a lot of garbage, however there was one interesting line

![](/assets/img/all/G8tCkxBI.jpg)

UPX, as shown in the screenshot, is a packer which is basically a compressor. Even though packed programs can still run without the need of decompressing, for static analysis we have to unpack it

![](/assets/img/all/G73Z3dfl.jpg)

Now we can analyze it in ghidra

![](/assets/img/all/TWjC65R4.jpg)

Of course we start with the `main` function. We find the string asking user for a password. Then we see that the output of the `exatlon` function is compared to what seems to be some encoded string. We can also see that even if we provide correct password the only thing we get is a *Looks Good* message, no flag is returned. Let's look at the `exatlon` function so hopefully we can decode the string, which seems to be the flag

![](/assets/img/all/F0Nh0vxr.jpg)

It looks really ugly because of the C++ syntax. Still we can identify how the mystery string is encoded

![](/assets/img/all/z_Fubd6-.jpg)

This is the most important part of the function. It iterates through the string provided and it **left** shifts each character (converted to int) by 4, which essentially means that the value is multiplied by 16 (2^4). So now that we found a probable encoding let's test it on the string we found

This simple python code shifts **right** by 4 each number in the string and converts it back to ASCII

```python
enc="1152 1344 1056 1968 1728 816 1648 784 1584 816 1728 1520 1840 1664 784 1632 1856 1520 1728 816 1632 1856 1520 784 1760 1840 1824 816 1584 1856 784 1776 1760 528 528 2000"

for c in enc.split(" "):
    d = int(c) >> 4
    print(chr(d), end="")

print()
```

![](/assets/img/all/zLrajhya.jpg)

HTB{l3g1c3l_sh1ft_l3ft_1nsr3ct1on!!}
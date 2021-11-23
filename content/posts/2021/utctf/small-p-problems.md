---
layout: post
title: Small P Problems - UTCTF 2021
date: 2021-03-15T00:00:00.000+10:30
description: ""
author: javad
categories:
  - ctf
  - write-ups
header_img: ""

---
The challenge description starts 'My buddies Whitfield and Martin were trying to share a secret key', so googling something like 'Whitfield Martin cipher' seems like a good place to begin. Immediately we get results for the [Diffie–Hellman key exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange), which fortunately can be described in terms of `A`, `B`, `p`, `g`, and `s` (the value of the secret key we need).

Scripts to brute-force this secret key are easy to find on GitHub. We used [this DHAttack.py script](https://github.com/zhangpengpengpeng/Diffie-Hellman-Algorithm) to get the flag `utflag{53919}`, although there are some much faster alternatives. Try this script to get the flag yourself:

```py
#!/usr/bin/python3

# Based on github.com/DrMMZ/Attack-Diffie-Hellman

p = 69691
g = 1001
A = 17016
B = 47643

for x in range(1, p):  # or (g**x) % p == B
if (g**x) % p == A:
a = x

s = (B**a) % p  # or (A**b) % p

print(s) 
```
---
layout: post
title: Non-textual Troubles - UACTF 2022
date: 2022-08-07T00:00:00.000Z
description: After messing up the implementation, I have decided that this is a feature and not a flaw.
author: javad
categories:
  - ctf
  - write-ups
  - crypto
---

It turn out that in Python 3, attempting to write non-ASCII characters to a file without using 'binary mode' (a mode which deals with 'non-textual data', hence the name of the challenge) has some less-than ideal results. Indeed, if you tried providing your own plain-text to `xor.py` you might have noticed that there are somehow more bytes in the cipher-text after XORing that you started with in your plaintext. Ultimately, it appears that the `write.write(ciphertext)` function is prepending either 0xc2 or 0xc3 to certain bytes. Simply adding a condition to exclude these, and providing the cipher-text as the input (since XOR is the inverse of itself) will provide a simple solution to this puzzle.

```py
from random import seed, randrange


seed(True, version=2)

with open("plaintext.txt", 'r') as read, open("ciphertext.txt", 'w') as write:
    plaintext = read.read()

    for char in plaintext:
        A = ord(char)
        if A != 194 and A != 195:  # exclude 0xc2 and 0xc3
            B = randrange(256)
            ciphertext = chr(A ^ B)
            print(bytes([A ^ B]))
            write.write(ciphertext)
```

**Flag**: `UACTF{b4d_h4b175_l34d_70_py7h0n2}`

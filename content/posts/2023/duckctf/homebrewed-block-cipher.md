---
layout: post
title: Homebrewed Block Cipher - DuckCTF 2023
date: 2023-08-04T00:00:00.000Z
description: I have finally created my own, completely secure, cipher. I am so confident that I will even allow you to encrypt your own data with it.
author:	lachlan 
categories:
  - ctf
  - write-ups
  - crypto
---

In this challenge, we are given an oracle that will encrypt our input with a constant key. We are also given a redacted version of the encrypting script. 

Reading through the script, we can see that data is encrypted by splitting the data into blocks of two characters, encrypting each block individually (with a redacted function), and then concatenating the output. Furthermore, by connecting to the oracle, we can see that each block gets encrypted to a fixed size of 40 characters. 

These observations make the encryption very weak, as if we make a table of what all pairs of characters get encrypted to, we can easily decrypt any encrypted message by splitting the encrypted message into blocks of 40 characters, and looking for the corresponding plaintext in our lookup table. 

The following Python script implements this exact solution

```python=
from pwn import *
from itertools import product

# ================= General Setup =================
conn = remote("chall.duckctf.com", 30002)
# Getting encrypted flag
encryptedFlag = conn.recvline()[16:].strip()
conn.recvline() # Cleaning new line

# Generating all possible pairs of characters in the characterSet
characterSet = list(string.ascii_lowercase + '0{}')
pairs = [p[0] + p[1] for p in product(characterSet, repeat=2)]


# ============ Creating a lookup table ============
# For speed, we just sent all pairs at once
# and split into blocks afterwards. However, we
# could theoretically send each pair one by one
payload = ''.join(pairs)
conn.sendline(bytes(payload, 'utf-8'))
enc = conn.recvline()[32:].strip()
encPairs = [enc[i:i+40] for i in range(0, len(enc), 40)]
table = dict(zip(encPairs, pairs))


# =============== Decrypting Flag ================
# Splitting the encrypted flag into blocks of 40
blocks = [encryptedFlag[i:i+40] for i in range(0, len(encryptedFlag), 40)]

# Searching for each block in lookup table
flag = ''.join([table[block] for block in blocks])
print(flag)
```

This gives the flag `quack{shortblockencryptionbad}`.

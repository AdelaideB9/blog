---
layout: post
title: A Based Crypto Challenge - DuckCTF 2023
date: 2023-08-04T00:00:00.000Z
description: There's not much to say here other than this challenge is **BASED**.
author:	lachlan 
categories:
  - ctf
  - write-ups
  - crypto
---

As hinted by the challenge title and description, this challenge is just some sort of base encoding. This is further confirmed by looking at the encoded data;

```
8990767883967987868C74768B8B90857A747A8678877981867C8B98
```

To determine the base, let us count the number of distinct symbols. 13! So this is likely to be base 13. There is of course a chance it was some other base where, by chance, the other symbols didn't get used, but let us first try base 13.

To decode base 13, we will need to make some assumptions about how the data was encoded.
1) The encoded data is regular ASCII -- i.e. `0 - 127`;
2) `0-9` in base 13 represents `0-9` in decimal, and `a-c` in base 13 represents `10-12` in decimal;
3) Each character has been encoded into base 13 individually, and the result has been concatenated with the other encoded characters.

From these assumptions, let us consider the minimum number of base 13 digits required to represent an ASCII character -- i.e. how many base 13 characters are needed to represent a number between 0 and 127? Well, one digit will give a range between 0 and 12, and two characters will range from 0 to 195 (as $14^2 - 1 = 195$). Hence we need at least 2 digits to represent all of the standard ASCII characters in base 13. Hence, if we assume that every two characters in the encoded data are one ASCII character, the first character of the encoded data is `89` in base 13, or `113` in decimal and thus `q` in ASCII. 

If we follow this process for all other pairs of characters in the encoded data, we get the flag

`quack{dont_assume_encodings}`

#### Cyberchef Usage
Decoding from an arbitrary base can be done on CyberChef through the use of the `From Charcode` ingredient.

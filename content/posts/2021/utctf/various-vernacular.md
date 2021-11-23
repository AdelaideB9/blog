---
layout: post
title: Various Vernacular - UTCTF 2021
date: 2021-03-15T00:00:00.000+10:30
description: ""
author: javad
categories:
  - ctf
  - write-ups
header_img: ""

---
We're given the encrypted flag `wmysau{foeim_Tfusoli}` along with some additional encrypted text to help us 'Hkgxologflutleiaymt xgf Azutgkrftmtf ltmntf ERW wfr ELW wfmtk Rkweq'.

Some familiarity with common ciphers, along with the hint 'This is a substitution cipher', give us a pretty good direction to pursue so we decided to use [this online tool](https://www.boxentriq.com/code-breaking/cryptogram "Substitution Cipher Solver Tool") for brute-forcing the solution.

However, trying brute-forcing the text rendered nothing more decipherable than the initial text. Instead we look to the challenge title, where we find that the word vernacular refers to local languages or dialects. Perhaps the cipher text is in a different language? The hint 'The plaintext may not necessarily be in English' confirms this.

Some experimentation with different languages and google translates produces the string in German 'provisionsgeschmüte von mageordneten setzen cdu und csu unter druck'. In English, 'disregarding commissions from the ordered people put cdu and csu under pressure.' Looks like we found a valid substitution! Trying (using all English characters) on the encrypted flag, with attention to capitalisation, gave us `utflag{nicht_English}`.

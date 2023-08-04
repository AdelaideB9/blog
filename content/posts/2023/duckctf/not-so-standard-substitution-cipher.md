---
layout: post
title: Not So Standard Substitution Cipher - DuckCTF 2023
date: 2023-08-04T00:00:00.000Z
description: I made a machine to implement a substitution cipher. The only issue is that it seemed to encrypt everything in sight, including my flag and all other random stuff. Each line in the attached file is a new piece of encoded information. Please save my flag! The flag will not be encased in `quack{...}`, but it will be the only reasonable text.
author:	lachlan 
categories:
  - ctf
  - write-ups
  - crypto
---

We are given a file with 10,000 lines, each of which is a new piece of data encrypted with a substitution cipher with a different key. One of these lines is the flag, and the rest are just random characters. 

To filter the rubbish out from the flag, we can use frequency analysis -- as a substitution cipher will not change the frequency of characters.

Ideally, we do not want to be comparing the character frequency distributions by hand. As such, we will want some way to rank each line in terms of their likelihood of being encrypted English -- i.e. some way to compare the frequency of English characters to the frequency of characters in each line. While there are numerous ways to do this, as the mathematician I am, I opted to compute the [Kullback–Leibler divergence](https://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence) for each line. This quantity is essentially a measure of the difference between two distributions and is given by the equation

$$D_{KL}(P||Q) = -\sum_{x} P(x) \log \left(\frac{Q(x)}{P(x)}\right)$$,

where $P$ and $Q$ are the probability distributions of interest. Hence, if we calculate the probability of each character occurring on each line and apply this formula along with the expected character frequency for English, we should see that the flag has the lowest divergence as it is the only data that is English. The following script does just that;

```python=
from collections import Counter
from math import log
import re

def getProbDist(s):
    n = len(s)
    freq = Counter(s)
    prob = [i/n for i in freq.values()]
    prob = sorted(prob, reverse=True)

    return prob

def KLD(P,Q):
    div = 0
    for i in range(min(len(P), len(Q))):
        div -= P[i] * log(Q[i] / P[i])

    return div

def main():
    # ==== Generating Probability Distribution From Sample Text ======
    with open('englishSample.txt') as f:
        englishSample = f.read().lower()

    # Stripping all but a-z characters
    englishSample = re.sub(r'[\[]|[^[a-z]]*', r'', englishSample)
    targetDist = getProbDist(englishSample)

    # Calculating Kullback–Leibler Divergence for each encrypted line
    rankings = {}

    with open('encrypted.txt') as f:
        for line in f:
            line = line.strip()
            dist = getProbDist(line)
            div = KLD(targetDist, dist)
            rankings[line] = div

    # Printing the most likely English string encrypted
    rankings = dict(sorted(rankings.items(), key=lambda item: item[1]))
    print(list(rankings.keys())[0])


if __name__ == "__main__":
    main()
```

Running this we find the most likely encrypted English string to be 
```
tgetrprgrpzvpttgtnkorpehkrzjfkdgkvnbyvyhbtptyvsikfkpttzmkjphhkfnzvrkvrrzpvnfkytkrikhkvxrizjrikjhyxpvhpvkyfyhxkefyriknybhkbiymphrzvrikzfkmvymksyjrkfrikmyrikmyrpnpyvtyfrigfnybhkbyvsqphhpymfzqyviymphrzvtryrktriyrkukfbtdgyfkmyrfplzukfynzmmgryrpukfpvxtgniytrikfkyhzfnzmohklvgmekftzfrikpvrkxkfttyrptjpktprtzqvniyfynrkfptrpnkdgyrpzv
```

Decrypting this using an [online tool](https://www.dcode.fr/monoalphabetic-substitution) gives us
```
substitutionissusceptibletofrequencyanalysisandhereissomefillercontenttoincreasethelengthoftheflaginlinearalgebrathecayleyhamiltontheoremnamedafterthemathematiciansarthurcayleyandwilliamrowanhamiltonstatesthateverysquarematrikoveracommutativeringsuchastherealorcompleknumbersortheintegerssatisfiesitsowncharacteristicequation
```
Which is indeed English text and the flag for the challenge.

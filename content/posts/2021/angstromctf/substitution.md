---
layout: post
title: Substitution - ångstromCTF 2021
date: 2021-04-08T13:12:52.939Z
description: "A write up for the ångstromCTF crypto challenge substitution "
author: lachlan
categories:
  - ctf
  - write-ups
---
For this challenge we are given a source file and a netcat server which presumably runs the source. Looking through the source code, we see that a integer is taken in as input and using this input, the flag is encrypted. The source is as follows:

```python
#!/usr/bin/python

from functools import reduce

with open("flag", "r") as f:
    key = [ord(x) for x in f.read().strip()]
    

def substitute(value):
    return (reduce(lambda x, y: x*value+y, key)) % 691


print("Enter a number and it will be returned with our super secret synthetic substitution technique")
while True:
    try:
        value = input("> ")
        if value == 'quit':
            quit()
        value = int(value)
        enc = substitute(value)
        print(">> ", end="")
        print(enc)
    except ValueError:
        print("Invalid input. ")
```

The main function of interest to us is the substitute function.

```python
def substitute(value):
    return (reduce(lambda x, y: x*value+y, key)) % 691
```

After researching what the reduce function does, we see that we are essentially recursively calling 

$$f(x,y) = kx+y$$

where $k$ is the user input, $x$ is the previous result and $y$ is the next character in the flag. As $f$ is a linear function, we can produce the following linear equation.

$$g(k) \equiv x_0 k^{n-1}+x_1 k^{n-2}+...+x_{n-2} k+x_{n-1} \pmod{691}$$

where $x_n$ is the $n$th character of the flag. To test our understanding, let us evaluate $g(0)$. As per $g(k)$, we should have $g(0)=x_{n-1}$. In other words, we should get the last letter of the flag, hence we should get the ASCII value of *}*. Connecting to the server and trying it, we indeed get $125$.

Knowing this, we can make a $n \times n$ linear system where the $n$th equation is the equation $g(n)$. 

$$\begin{aligned}g(0) &\equiv x_0 \times 0^{n-1}+ x_1 \times 0^{n-2}+ ... +x_{n-2} \times 0+x{n-1} \pmod{691} \\\ g(1) &\equiv x_0 \times 1^{n-1}+ x_1 \times 1^{n-2}+ ... +x_{n-2} \times 1+x_{n-1} \pmod{691} \\\ g(2) &\equiv x_0 \times 2^{n-1}+ x_1 \times 2^{n-2}+ ... +x_{n-2} \times 2+x_{n-1} \pmod{691} \end{aligned}$$

This can be expressed as:

$$A\textbf{X}=\textbf{B} \pmod{691}$$

where A is the coefficient matrix:

$$\begin{bmatrix}0^{n-1} & 0^{n-2} & \cdots & 0^{1} & 1 \\\ 1^{n-1} & 1^{n-2} & \cdots & 1^{1} & 1 \\\ \vdots & \vdots & \ddots & \vdots & \vdots  \\\ (n-2)^{n-1} & (n-2)^{n-2} & \cdots & (n-2)^{1} & 1 \\\ (n-1)^{n-1} & (n-1)^{n-2} & \cdots & (n-1)^{1} & 1\end{bmatrix}$$

and B is the outputs for each given input:

$$\begin{bmatrix}g(0) \\\ g(1) \\\ \vdots \\\ g(n-2) \\\ g(n-1) \end{bmatrix}$$

and $X$ is the matrix of characters (essentially the flag).

While we are not sure as to how long the flag will be, it is reasonable to say it will be less that 100 characters (based off previously retrieved flags). Using this equation and the fact that the characters must be integers, we can solve the system over $\mathbb{Z}\pmod{691}$. This can be done with the following sage maths script that uses *pwntools* to connect to the server and collect the results.

```python
from pwn import *
import re

# Connecting with pwntools
nc = remote('crypto.2021.chall.actf.co', 21601)
nc.recvline()

nums = []

for i in range(100):

    # Sending the numbers from 0-99 to the server and listening to the response
    nc.sendline(str(i))
    res = str(nc.recvline())
    # Stripping the number from the response and appending it to numbers
    nums.append(int(re.findall("[0-9]+", res)[0]))

# Using sagemaths to solve the linear system
M = matrix(ZZ, 100, 100, lambda x, y: pow(x, 99-y))
b = vector(GF(691), 100, nums)
solution = M.solve_right(b)

# Converting the solution into a string
flag = ''.join(chr(c) for c in solution)

print(flag)
```

Sure enough, after a minute or so of running, we get the flag!

*actf{polynomials_20a829322766642530cf69}*

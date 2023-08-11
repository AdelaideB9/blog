---
layout: post
title: Evil Eval - UACTF 2022
date: 2022-08-07T00:00:00.000Z
description: Made a cool internet calculator for all your maths needs. Well, some of your maths needs.
author: javad
categories:
  - ctf
  - write-ups
  - pwn
---

Trying a variety of inputs over netcat, you'll quickly discover two key pieces of information from the error messages:

1. The characters 'f', 'l', 'a', 'g', '.', 't', 'x', 't', and '`' are all blocked
2. Our input can't have more than eight distinct characters

We can infer that our goal is something to the effect of making a system call like `cat flag.txt` in eight or fewer characters. Looking through [Ruby's pre-defined variables](https://ruby-doc.org/docs/ruby-doc-bundle/Manual/man-1.4/variable.html), we can see that `$"` denotes a long list of module names (loaded by require) which we can potentially character index and frankenstein together to write out "flag.txt" in relatively few distinct characters, like so:

```
f -> $"[11+1+1+1][11+11+11]
l -> $"[1+1+1][1+1+1+1]
a -> $"[1+1][1]
g -> $"[11][11+11+1+1]
. -> $"[1][1+1+1+1+1+1]
t -> $"[1+1][1+1]
x -> $"[1+1+1][1+1+1+1+1+1]
t -> $"[1+1][1+1]
```

The next step is to get Ruby to print the contents of this file. One trick is to pass `ARGV << flag.txt`, which will cause our ruby script to call itself, passing the evaluated contents of `flag.txt` to itself, exposing the flag. Of course, this requires two many distinct characters, but by referring back to our trusty pre-defined variables, we see that the much shorter `$*` is an alias to `ARGV`. Putting all this information, we can create the following 'payload' to expose the flag:

```
$*<<$"[11+1+1+1][11+11+11]+$"[1+1+1][1+1+1+1]+$"[1+1][1]+$"[11][11+11+1+1]+$"[1][1+1+1+1+1+1]+$"[1+1][1+1]+$"[1+1+1][1+1+1+1+1+1]+$"[1+1][1+1]
```

**Flag:** `UACTF{8u7_53210u51y_d0n7_3v41}`

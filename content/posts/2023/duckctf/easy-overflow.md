---
layout: post
title:  Easy Overflow - DuckCTF 2023
date: 2023-08-04T00:00:00.000Z
description: I store all of my private data in all of my programs. I mean, why not? It is safe, right...
author:	lachlan 
categories:
  - ctf
  - write-ups
  - pwn 
---

We have been provided with the C code for this challenge;

```C=
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    int id = 0;
    char name[16] = "";

    printf("Input your name: ");
    gets(name);
    printf("Your name is %s with ID %d.\n", name, id);

    if (id == 1179402567) {
        printf("%s\n", argv[1]);
    }

    return 0;
}
```

As we can see, we are using the vulnerable `gets` function. We can use `gets` to overwrite the `id` variable which is just above the `name` variable on the stack. To do this, we need to write 16 bytes into `name` and then our desired value into `id` -- which is `1179402567`. This is easy enough to do. The only friction here is writing `1179402567` into `id` as we must input the ASCII representation for `1179402567`. However, by converting `1179402567` into ASCII we can see that `1179402567` is equal to the ASCII string `FLAG`, which is easy enough to enter by hand. The final potential problem arises from the fact that strings are written to the stack starting at the lowest memory address, and ending at the highest. This is problematic as this binary is little endian, which means that we will have to enter `FLAG` backwards -- again, this is easy to do. This gives our final payload of:
```
aaaaaaaaaaaaaaaaGALF
```

Entering this into the running binary gives us the flag `quack{gets_is_vulnerable}`.

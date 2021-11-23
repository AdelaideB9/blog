---
layout: post
title: Chicken - UMassCTF '21
date: 2021-03-29T01:30:00
author: samiko
categories:
- ctf
- write-ups
header_img: ''
description: 'Chicken Chicken Chicken: Chicken Chicken? A forensics category challenge all about extracting hidden streams in a PDF file and 7-Zip password cracking.'

---
## Investigating the mystery PDF File

We're given a modified PDF file of the infamous research paper, "Chicken Chicken Chicken: Chicken Chicken", by Doug Zongker at the University of Washington.

[chicken.pdf](https://www.notion.so/signed/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F1265e1d3-f480-4881-a1f6-b188cdc8701e%2Fchicken.pdf?table=block&id=217d1ee6-a5ff-47aa-9518-b9cd4305949d)

![https://i.imgur.com/jBSJVNH.png](https://i.imgur.com/jBSJVNH.png)

Since we know this is a published research paper, we can download a copy of the [original PDF file](https://isotropic.org/papers/chicken.pdf) and compare the two for any difference:

![https://i.imgur.com/bE2dmRu.png](https://i.imgur.com/bE2dmRu.png)

We see that at around line 202, there is an extra OpenAction object inserted into the document, with a data stream beginning with `7z`:

`$ hexdump -C chicken.pdf | grep 7z -A 11`

```
00001980  0d 0a 37 7a bc af 27 1c  00 04 2b 65 00 6c 30 00  |..7z..'...+e.l0.|
00001990  00 00 00 00 00 00 6a 00  00 00 00 00 00 00 4c 6a  |......j.......Lj|
000019a0  b9 1e 0c fd be 4f 3b 93  39 58 52 bd 23 ea 0b 2d  |.....O;.9XR.#..-|
000019b0  8d d1 a2 79 55 0b d8 05  68 43 0d ae 06 d5 2d f8  |...yU...hC....-.|
000019c0  25 ff b4 16 8d 21 3b 88  16 35 44 69 6d 5c 0e 59  |%....!;..5Dim\.Y|
000019d0  a7 b3 01 04 06 00 01 09  30 00 07 0b 01 00 02 24  |........0......$|
000019e0  06 f1 07 01 0a 53 07 56  f2 43 9d 21 42 28 ae 21  |.....S.V.C.!B(.!|
000019f0  21 01 00 01 00 0c 29 25  00 08 0a 01 4e 5d 1c 8e  |!.....)%....N]..|
00001a00  00 00 05 01 19 09 00 00  00 00 00 00 00 00 00 11  |................|
00001a10  0f 00 73 00 65 00 63 00  72 00 65 00 74 00 00 00  |..s.e.c.r.e.t...|
00001a20  19 04 00 00 00 00 14 0a  01 00 80 33 4a 2c b7 1d  |...........3J,..|
00001a30  d7 01 15 06 01 00 20 80  a4 81 00 00 0a 65 6e 64  |...... ......end|
```

The data stream starts with the 7z magic bytes, confirming that it is indeed a 7z file:

[List of file signatures - Wikipedia](https://en.wikipedia.org/wiki/List_of_file_signatures)

![https://i.imgur.com/99riFle.png](https://i.imgur.com/99riFle.png)

Let's extract the stream with a bit of Bash-fu to a `chicken.hex` file:

`$ hexdump -C chicken.pdf | grep 7z -A 11 | cut -d ' ' -f3- | rev | cut -d ' ' -f3- | rev > chicken.hex`

```
0d 0a 37 7a bc af 27 1c 00 04 2b 65 00 6c 30 00
00 00 00 00 00 00 6a 00 00 00 00 00 00 00 4c 6a
b9 1e 0c fd be 4f 3b 93 39 58 52 bd 23 ea 0b 2d
8d d1 a2 79 55 0b d8 05 68 43 0d ae 06 d5 2d f8
25 ff b4 16 8d 21 3b 88 16 35 44 69 6d 5c 0e 59
a7 b3 01 04 06 00 01 09 30 00 07 0b 01 00 02 24
06 f1 07 01 0a 53 07 56 f2 43 9d 21 42 28 ae 21
21 01 00 01 00 0c 29 25 00 08 0a 01 4e 5d 1c 8e
00 00 05 01 19 09 00 00 00 00 00 00 00 00 00 11
0f 00 73 00 65 00 63 00 72 00 65 00 74 00 00 00
19 04 00 00 00 00 14 0a 01 00 80 33 4a 2c b7 1d
d7 01 15 06 01 00 20 80 a4 81 00 00 0a 65 6e 64
```

From reading the [technical specifications](https://www.7-zip.org/recover.html) of the 7z file format, we know the file has to begin with `37 7A BC AF 27 1C` and end with `00 00`. Therefore, we can trim off the starting `0D 0A` and the ending `0A 65 6E 64` bytes, as they are not a part of the file.

Convert the hex dump to a 7z file:

`$ xxd -r -p chicken.hex chicken.7z`

## Extracting and cracking 7z password hash with John the Ripper

Upon trying to extract the 7z file, we're greeted with a password prompt:

`$ 7z x chicken.7z`

    7-Zip [64] 17.03 : Copyright (c) 1999-2020 Igor Pavlov : 2017-08-28
    p7zip Version 17.03 (locale=en_AU.UTF-8,Utf16=on,HugeFiles=on,64 bits,8 CPUs x64)
    
    Scanning the drive for archives:
    1 file, 186 bytes (1 KiB)
    
    Extracting archive: chicken.7z
    --
    Path = chicken.7z
    Type = 7z
    Physical Size = 186
    Headers Size = 138
    Method = LZMA2:12 7zAES
    Solid = -
    Blocks = 1
    
    Enter password (will not be echoed): _

We can obtain the password hash with `7z2john`:

`$ 7z2john ./chicken.7z > chicken.hash`

    chicken.7z:$7z$2$19$0$$8$56f2439d214228ae0000000000000000$2384223566$48$41$0cfdbe4f3b93395852bd23ea0b2d8dd1a279550bd80568430dae06d52df825ffb4168d213b88163544696d5c0e59a7b3$37$00

Now that we have the password hash, let's crack it using John with the rockyou.txt wordlist:

`$ john chicken.hash --wordlist=rockyou.txt --format=7z-opencl`

    Device 2@arch-zippy: GeForce GTX 1070
    Using default input encoding: UTF-8
    Loaded 1 password hash (7z-opencl, 7-Zip [SHA256 AES OpenCL])
    Cost 1 (iteration count) is 524288 for all loaded hashes
    Cost 2 (padding size) is 7 for all loaded hashes
    Cost 3 (compression type) is 2 for all loaded hashes
    Will run 8 OpenMP threads
    Press 'q' or Ctrl-C to abort, almost any other key for status
    0g 0:00:00:15 0.36% (ETA: 00:59:49) 0g/s 3576p/s 3576c/s 3576C/s Dev#2:61°C iiloveyou..simone13
    0g 0:00:11:47 16.93% (ETA: 00:59:03) 0g/s 3736p/s 3736c/s 3736C/s Dev#2:59°C yahoomylove..y2j341
    0g 0:00:19:51 29.15% (ETA: 00:57:34) 0g/s 3643p/s 3643c/s 3643C/s Dev#2:60°C rebel9250..rdoleo
    pineapple95 (chicken.7z)
    1g 0:00:21:25 DONE (2021-03-29 00:10) 0.000778g/s 3627p/s 3627c/s 3627C/s Dev#2:59°C pinkice88..pincy
    Use the "--show" option to display all of the cracked passwords reliably
    Session completed

After 21 gruelling minutes, we get the cracked password:

`pineapple95`

Extract the 7z:

`$ 7z x chicken.7z`

    7-Zip [64] 17.03 : Copyright (c) 1999-2020 Igor Pavlov : 2017-08-28
    p7zip Version 17.03 (locale=en_AU.UTF-8,Utf16=on,HugeFiles=on,64 bits,8 CPUs x64)
    
    Scanning the drive for archives:
    1 file, 186 bytes (1 KiB)
    
    Extracting archive: chicken.7z
    --
    Path = chicken.7z
    Type = 7z
    Physical Size = 186
    Headers Size = 138
    Method = LZMA2:12 7zAES
    Solid = -
    Blocks = 1
    
    Enter password (will not be echoed): pineapple95
    Everything is Ok
    
    Size:       37
    Compressed: 186

The 7z file contains a `secret` file, let's read it:

`$ cat secret`

    VU1BU1N7QF9sIUxfNW03SCFuXzN4N3JAfQo=

This string seems to be encoded in base64, as hinted by the `=` padding. Decoding it gives:

`$ echo "VU1BU1N7QF9sIUxfNW03SCFuXzN4N3JAfQo=" | base64 -d`

    UMASS{@_l!L_5m7H!n_3x7r@}

Winner winner chicken dinner, we got the flag!

## Resources

1. [https://isotropic.org/papers/chicken.pdf](https://isotropic.org/papers/chicken.pdf)
2. [http://myexperimentswithmalware.blogspot.com/2014/09/pdf-analysis-with-peepdf.html](http://myexperimentswithmalware.blogspot.com/2014/09/pdf-analysis-with-peepdf.html)
3. [https://en.wikipedia.org/wiki/List_of_file_signatures](https://en.wikipedia.org/wiki/List_of_file_signatures)
4. [https://www.7-zip.org/recover.html](https://www.7-zip.org/recover.html)
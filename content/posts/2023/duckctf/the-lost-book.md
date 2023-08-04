---
layout: post
title: The Lost Book - DuckCTF 2023
date: 2023-08-04T00:00:00.000Z
description: I run a library, but recently one of my books was returned damaged. Can you please find the book title? 

*The flag for this challenge is the title of this book in the original language.*
author:	lachlan 
categories:
  - ctf
  - write-ups
  - osint
---

This challenge only provides the following image of a book cover;
![](https://hackmd.io/_uploads/SJr3_Bqin.jpg)

As we can see, the ISBN is partially corrupted, and the goal is to recover the ISBN and thus recover the book title. 

After some quick [googling](https://en.wikipedia.org/wiki/ISBN#Overview), the structure of an ISBN10 code can be found. The following information is relevant;
    
1) The first digits represent the country of publication,
2) the next few digits represent the publisher,
3) the remaining digits except the final digit specify the book's title and edition,
4) and the final digit is a checksum.

While the specific number of digits in each section varies on the size of the publisher, the country, etc, we can still use this information and the corrupted ISBN provided to recover the book.

First, we can see that the first digit is missing; thus we need to identify the country of publication. As the characters on the book cover are Japanese, it is safe to assume the country is Japan, which has a code of `4`. Thus the first character of the ISBN in `4`.

Next, we need to determine the publisher and their ISBN publisher code. For this, Google Lens comes to our aid! Scanning the book, we can tell that the publisher is [`オーム社 (ohmsha)`](https://www.ohmsha.co.jp/). Searching for this and going to the Japanese publications, we can see that they have published a large variety of books (hence brute-forcing it from here will not work). However, as the publisher code in ISBNs is mostly constant, we can just select any book published by them, and copy the publisher code in the ISBN of the randomly selected book. Doing this will give us the publisher code of `274`. This `2` in `274` aligns with the `2` already given to us in the ISBN: which is a good sign!

Finally, we need to determine the last missing digit. While this could be brute forced, we can also use the checksum in the ISBN (final digit) to recover the final missing digit. While I will not go into details of this here, the checksum calculation is clearly explained in the [Wikipedia article for ISBNs](https://en.wikipedia.org/wiki/ISBN#ISBN-10_check_digits), and reversing it to recover a single digit is a simple exercise in algebra. Doing this gives the final digit of `7`. 

Putting all the digits together gives us the final ISBN of `4274066746`. Searching this up on an [ISBN lookup](https://isbnsearch.org/) will give us the book (and flag) `マンガでわかる暗号`, which in English is `Cryptography Understood by Manga`.

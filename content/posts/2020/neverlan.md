---
layout: post
title: NeverLAN CTF 2020 Write-up
date: 2020-02-08T14:30:10.000+00:00
author: javad
categories:
- ctf
- write-ups

---
## Browser Bias

This challenge gives us very little information, just a url to a site that tells us `Sorry, this site is only optimized for browsers that run on commodo 64`. However, this also narrows our focus down to a singular goal - trying to convince the website that we are accessing it from a whatever a 'commodo 64' is.

The first thing we need to know is how the browser can determine what type of client is making a request to it. It turns out, a site can only determine this through the `User-Agent` header. We can manipulate this information in a variety of ways - like with browser extensions or on some browsers using the Developer Tools - but for the sake of controlling exactly what headers are passed over we made this handy little python function.

{% gist 889226036803bab244ae41b127dc0ab3 %}

Next we have to figure out what our user-agent string should actually be. If we pass 'commodo 64' as the user-agent parameter of our function nothing changes. Googling commodo 64, we get results for the 1982 computer, The Commodore 64, but making that the header doesn't work either. The part of the challenge we're forgeting is that the site requires `browsers that run on commodo 64`. We need to find the header of a browser that works on the Commodore 64 ...

... here's one! `Hyperlink/2.5e (Commodore 64)`

Put that into our function and we get `Welcome fellow c64 user. flag{8b1t_w3b}`

## Robot Talk

We found the best way to comunicate with [challenges.neverlanctf.com:1120](challenges.neverlanctf.com:1120 "challenges.neverlanctf.com:1120") was through Telnet!

Here's the handy script we made ...

{% gist 5edbe8981351b9213bad90cd38135a0d %}

... and here's the outputted flag `flag{Ant1_hum4n}`

## BitsnBytes

Given the 'programming' category, the title, and the fact that we are only linked to an image with green and gray squres, we figured that each colour represented a bit (`1` or `0` in binary) and we had to write a script that would translate it.

{% gist 24c0d9afe556dba4f84e383b62d4469b %}

Here, each green square is decoded as a zero, and each gray is a one.  Since this is an svg, which is conveniently a file type where infomation is stored in plain text, we didn't need to bring out any python image-processing libraries. Then we would convert the binary string to plain text.

The problem was, we weren't getting the flag - we were getting something like this `time hash:076a1b24abb4870fc5eb892de796b5b32642f95664c04ff0d9b2b96c58a0b`. The trick to this challenge was that the server only spits out the image of the encoded flag at hourly intervals, during the minute after the clock has shifted to the new hour. Once we got [the right image](/assets/uploads/php.svg) we also got `Now you've got it! Here's your flag: flag{its_all_ab0ut_timing}`.

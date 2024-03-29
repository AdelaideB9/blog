---
layout: post
title: Blurry-Eyed - UACTF 2022
date: 2022-08-07T00:00:00.000Z
description: I don't know about you, but I like my images in stereo.
author: javad
categories:
  - ctf
  - write-ups
  - misc
---

Based on the description, you may have determined that we are dealing with an [autostereogram](https://en.wikipedia.org/wiki/Autostereogram), better known as a [magic eye](https://en.wikipedia.org/wiki/Magic_Eye) puzzle. As such, you can theoretically just stare at the picture with great intensity until the flag reveals itself to you. If you did manage to solve this challenge with only your eyes then you are amazing. Discerning simple shapes is difficult, let alone a short sentence.

If you are a mere mortal, an alternative way to solve this challenge is to use any of the online stereogram-solving tools that exist ([this one's](http://magiceye.ecksdee.co.uk/) pretty good). Alternatively, you can open up GIMP and put the image on two layers. Then set the blending mode of the top layer to 'difference' and drag the top layer along the horizontal axis until you get something like this: 

![Gimp Decoded Image](./assets/decoded.png)

Regardless of what solution you use, the shape of the letters should bear a resemblance to the original image, shown below.

![Original Text](./assets/original.png)

**Flag:** `UACTF{r34l17y_1n_3d}`
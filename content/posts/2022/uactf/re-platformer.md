---
layout: post
title: re-platformer - UACTF 2022
date: 2022-09-19T00:00:00.000Z
description: Have a look around the map for anything that might be of help. It won't be easy.
author: 0xRF
categories:
  - ctf
  - write-ups
  - reversing
---

This challenge was created with the intention of showing partipicants how easy it is to decompile .NET code,  presented in a fun game challenge. 

The challenge provides a Unity game. This is evident upon launching the game where you're greeted with "Made with Unity".

Exploring the game there's a jump that cannot be made. Rereading the descrption "*Have a look around the map for anything that might be of help. It won't be easy.*" It's evident the challenge is to somehow manipulate the players movement capabilties.
![](20220918165442.png)

Investigating the game files and with a quick google search Unity games reveals, the game is most likely built with C#.
![](20220918165257.png)

With a tool such as DnSpy it's possible to reverse and modify the C# code.

We need to know figure out where exactly the game code is stored. A quick search for Unity game hacking reveals, it's stored in the Managed folder. 
Navigating into `RE Platformer_Data\Managed` there's a bunch of DLLs, the game code is contained within `Assembly-CSharp.dll`.
	
![](20220918170524.png)

Opening this file up in DnSpy reveals `PlayerMovement`, this is exactly what we're looking for.
![](20220918170648.png)

To modify the game behaviour, the easiest way to go about it is within the start function. This function is called by the game engine once the object is created.  
![](20220918170757.png)
1. Right clicking
2. 'Edit Method'
3. Modify the gravityScale to say 5% of the original
  ![](20220918171341.png)

Then saving the modified file and replacing the orignal one.
![](20220918171036.png)

With this we're able to jump our way to the flag.

**Flag**: `UACTF{ALPH4B3T_K1NG_4PPL3}`
![](20220918171320.png)
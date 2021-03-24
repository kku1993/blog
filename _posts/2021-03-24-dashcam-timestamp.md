---
layout: post
title:  "Simple Counter Could Save You 100% or More on Car Repair"
date:   2021-03-24
categories: tech
---

My dashcam has an unfortunate design flaw that caused it to fail to record a
recent accident. More frightening is how easy it is for anyone to make the same
mistake.

# TL;DR

Whenever possible, use a *monotonically increasing counter* to determine the
relative age of data stored on persistent storage, especially if your code
runs on devices that may lose power and have its clock reset.

# Background

I was driving down a highway when something fell off of a car in front of me
and damaged my bumper. It was not a serious accident and my car suffered only
cosmetic damage.

After I pulled off of the highway, I looked for footage from my dashcam to
identify the car that is responsible. To my surprise, I was not able to find
the relevant footage.

# Debugging the Dashcam

Right after the accident, I noticed that while the SD card is full of footages,
most videos are from 3 months ago while there is one file from 1970. The list of
files looked something like:

```
VID_19700121_083000.mp4
...
VID_20201221_175800.mp4
VID_20201221_180100.mp4
```

The file from 1970 contains footage from moments right after the accident, and I
noticed that the current time displayed on the dashcam is in 1970.

I also know that the dashcam records things in 3 minute segments and stores them on
the SD card. When the SD card is full, it overwrites the "oldest" file.

Therefore, I believe the dashcam recently lost power and had its clock reset to
Unix epoch. But since the SD card is filled with footages from 2020, the dashcam
was overwriting the same file from 1970 over and over again.

# Recommended Solution

This problem illustrates yet another example of how hard it is to deal with time
in software. In this particular case, a better approach would have been to use
a *monotonically increasing counter* to determine the "age" of each file. Even
if the dashcam loses power and the counter is reset to zero, at least you
wouldn't run into the problem where the "oldest" file is always the same.

P.S. Thank you [GEICO](https://youtu.be/-3oONClGBd8) for inspiring the title of
this post!

P.S. This post is not sponsored by GEICO.

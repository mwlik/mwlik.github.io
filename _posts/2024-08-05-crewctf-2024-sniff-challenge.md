---
layout: post
title: CrewCTF 2024 Sniff One, Sniff Two Writeup
subtitle: A Two Parts Hardware Challenge
mathjax: true
cover-img: https://github.com/user-attachments/assets/4515ddb7-b48d-4a6a-b583-8bfdcc101dc8
thumbnail-img: https://github.com/user-attachments/assets/4515ddb7-b48d-4a6a-b583-8bfdcc101dc8
share-img: https://github.com/user-attachments/assets/4515ddb7-b48d-4a6a-b583-8bfdcc101dc8
tags: [CTFs Writeups, Hardware]
author: M411K
---

## Sniff One = 27 solves, Sniff Two = 16 solves

<img width="350" alt="image" src="https://github.com/user-attachments/assets/419aaaca-70a6-421a-a535-995fb2372d8c">
<img width="350" alt="image" src="https://github.com/user-attachments/assets/3236cfd0-47e4-409b-9c02-84f491f87118">

## Overview

This was a two parts hardware challenge, we were provided with an attachment [dist.zip](https://github.com/user-attachments/files/16499353/dist.zip), that contains a [README.pdf](https://github.com/user-attachments/files/16499323/README.pdf) and a `.sal` capture (check the attachment).

It was a setup linking a cardKB mini keyboard and an e-paper display with Raspy 4 model b:

<img width="350" alt="image" src="https://github.com/user-attachments/assets/89ed837a-b018-4c18-b051-da7b03ddfa49">
<img width="350" alt="image" src="https://github.com/user-attachments/assets/fdf39b01-edfc-4f45-b77f-eb7acc48093f">

```
This challenge has two flags in the flag{} format:
  - The first (easier) is the password that was typed on the keyboard.
  - The second (significantly harder) is what was displayed to the screen after the password was entered.
```

So the challenge was depicting a person that entered the password (first easier flag) then the flag (second much harder flag) was displayed on the display, while everything was [sniffed](https://en.wikipedia.org/wiki/Sniffing_attack) by the [salae logic analyzer](https://www.saleae.com/), thus the job is to decode everything that was recoded to what it represent either its what was entered by the keyboard, or what was displayed in the e-ink display.

## Sniff One

So to find the first flag we need to find what was typed by the user of the keyboard, after researching the keyboard that was used, i found this pretty good [reference](https://docs.m5stack.com/en/unit/cardkb), and in it we find that the Communication method is [I2C](https://en.wikipedia.org/wiki/I%C2%B2C), for those that don't know how I2C there is some good references u can consult, one of these is this [video](https://youtu.be/CAvawEcxoPU?si=dEOXzUjc35swLupM), so I2C is a communication protocal that has generally two channels `SDA` (the one responsiblle for transmitting data), `SCL` (the clock that synchronize the communication):

<img width="400" alt="image" src="https://github.com/user-attachments/assets/4bf58ea0-7e35-4729-8fa1-5ba62a7b9e66">

And looking at the sal capture:

<img width="400" alt="image" src="https://github.com/user-attachments/assets/2ad141d7-2dbb-40fb-be76-1b47a4420224">

I figured that D0 and D1 is SDA and SCL respectively, but you can verify this looking at pictures of the wiring like this one:

<img width="400" alt="image" src="https://github.com/user-attachments/assets/7ea5953b-e470-4cf3-8dcc-6615ec8c8775">

So I added the I2C analyzer to [Logic 2](https://www.saleae.com/pages/downloads) and exported the data as csv.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/5e2c66b9-928e-405f-9482-c1a2efeabe95">

Now its time to decode those signals, but instead of doing it by hand why not just find a libray an already implemented library that will handle that for me?

That's exactly what I did and found [this one](https://github.com/ian-antking/cardkb), so I made some modification to the code to suit my use case:

<!-- TODO: ADD THE SOLVE SCRIPT -->

and tada!

```
KEY_F
KEY_L
KEY_A
KEY_G
KEY_LEFTSHIFT KEY_LEFTBRACE
KEY_7
KEY_1
KEY_7
KEY_F
KEY_7
KEY_5
KEY_3
KEY_2
KEY_LEFTSHIFT KEY_RIGHTBRACE
KEY_ENTER
```

flag: `flag{717f7532}`

## Sniff Two


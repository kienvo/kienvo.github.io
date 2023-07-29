---
layout: post
title: U-Boot stuck on Allwinner H616 and axp313a
date: 2023-07-29 20:02 +0800
description: 'This is a short note to fix the stuck issue when working with axp313a..'

categories: [sbc]
tags: [axp313a, sunxi50, h616, allwinner, u-boot, mcore-h616]
---

## TL;DR

The fix is [here](https://bbs.aw-ol.com/topic/2054/mq-quad-h616-%E4%B8%BB%E7%BA%BF%E5%86%85%E6%A0%B8%E7%BC%96%E8%AF%91%E8%B0%83%E8%AF%95%E8%AE%B0%E5%BD%95-u-boot-kernel-buildroot?_=1690635256062&lang=en-US) if you lazy.

## Note

I have used Orange Pi Zero2 for another project previously. It is used Armbian
pre-built image. Recently, I been working in a project using `mcore-h616`.
mcore-h616 have similar design with MQ-Quad. The image file produce by Mango Pi
website for MQ-Quad is working perfectly with mcore-h616. But I just want a
simple Linux system. I'm starting to build my own using Buildroot. And nothing
is easy, when I plug in the SD card, turn on the power. SPL give me some error:

``` log
U-Boot SPL 2023.04 (Jul 29 2023 - 18:27:29 +0700)
Failed to set core voltage! Can't set CPU frequency
DRAM: 128 MiB
Trying to boot from MMC1 # Sometime it could make it here and the stuck
```

My board have a 1 GiB ram but it show 128 MiB and some time changed to some
random power-of-2 number. I thought it could be DRAM config or PMIC in dts config
is wrong. It took me almost a week to try to change config in uboot config and
dts. I have searched all Google, DuckDuckGo,.. trying to read everything to
config my dts for axp313. Nothing in English article could help me.

Then to day, I thought I would copy 1 MiB section of MQ-Quad image to just get
it done and give up. But what is the meaning of the line `pmic id is 0x4b` when
boot success. I tried Googling with quote "pmic id is". The [last
result](https://bbs.aw-ol.com/user/evler) (of 11) got me to a Chinese website.

The user `evler` have a article to build u-boot, kernel and buildroot for
MQ-Quad. I tried to change `drivers/power/axp305.c` to his instruction. It run
perfectly. The detail of the code, I will update later (maybe).

## Documentation and Google Search

None of the English document was said about this. Every forum with the same
error with no anser. I thought I would write this in Vietnamese but then there
is no English docs is said about this specific problem.

It seem like Google don't have good search results with Chinese products.
Obiviously, Google didn't do bunsiness in China. I thought learn Chinese will
easier in solving problem like this, and the same with Russian. "know the enemy
and know yourself"
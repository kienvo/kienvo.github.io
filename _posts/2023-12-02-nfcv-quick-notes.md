---
layout: post
title: "NFC-V: quick notes"
description: >
  Here is some notes when I've been working on NFC recently. 
  I hopes this will help someone who just like me being confused between NFC's
  generations and standards.
date: 2024-12-02 20:19

image: '/assets/nfc-quick-notes/nfc-logo.png'

tags: [NFC-V, NFC, ST25DV]
categories: [notes]
---

A NFC tag is a chip inside a NFC card. Dynamic NFC tag is a chip that can
communicate with MCU. A NFC reader is a chip that read tags, and sometimes
comunicate with other reader, which is called NFC Emulators. Some types of cards
are emu-able, some are not.

NDEF is a standard on top of NFC-A, NFC-B, and NFC-F. It's a nice protocol
standdard but if you've got a ST25DV, it cannot be used with NDEF. ST25DV follow
ISO15693 standard (or NFC-V).

On Android, to comunicate with NFC-V tags, use [nfc-v
tranceive()](https://developer.android.com/reference/android/nfc/tech/NfcV#transceive(byte[])),
with the input is raw ISO15693 bytes.

![Tuxedo Winnie The Pooh Meme of NFC-V](/assets/nfc-quick-notes/pooh-nfcv.png)
_The feeling of being elite while calling something by its standard instead of name_

You can't affort to by ISO15693 standard? No worries, stick to the chip's
datasheet/manual, here is my example with
[ST25DV](https://www.st.com/resource/en/datasheet/st25dv04kc.pdf). In section
7.4, all (maybe) ISO15693 commands are listed there, follow some custom (vendor)
commands.

ST25DV's default password is 00000000 - 8 bytes of zeros. To open a security
session, use Present Password command. To close one, present a wrong password.

ISO15693 define the length of Read/Write Messages is `the length actually
transferred` = `the length that you put in raw message` + 1. And the maximum
data length to be transferred is 256. But 256 won't fit in a byte. So that is
why `the length that you put in raw message` is subtracted by one.

When using `react-native-nfc-manager`, call `requestTechnology()` once, then
doing the tranceive as much time as you want. I've been struggling of failing
requestTechnology() while calling it multiple times.

Some NFC tag claim to be dynamic but it lack of SRAM section to comunicate
between a reader and MCU such as M24LR. It is a dynamic tag, but frequently
writing to EEPROM is not a good choice. Imagine your product broke after 1000
uses. That is why ST25DV came to replace M24LR. It has a section of memory call
FTM. This can be used to transfer messages between MCU and readers.

I've been using this to transfer bitmaps to an epaper diplay. Some initial
thought that it would need a MCU with big size of SRAM to carry bitmaps for the
display. But why we store bitmaps on MCU while we pass every messages on NFC
tags to the display?

## References

[ST25 NFC guide](https://www.st.com/resource/en/technical_note/tn1216-st25-nfc-guide-stmicroelectronics.pdf)
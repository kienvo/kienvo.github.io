---
layout: post
title: "FOSSASIA Badge Magic Open Firmware"
description: 'Google Summer of Code 2024 - Final Submission Report'
date: 2024-10-07 10:27

image: '/assets/fossasia-badge-magic-open-firmware-google-summer-of-code-2024/GSoCxFOSSASIA.svg'

tags: [Google Summer of Code 2024, GSoc2024, GSoc'24, GSoC, FOSSASIA, Badge Magic, BLE, USB, HID, CDC ACM]
categories: [projects]
---

- Organization: [FOSSASIA](https://fossasia.org/).
- Mentors: [Francois Cartegnie](https://github.com/fcartegnie), [MarioB](https://github.com/mariobehling), [Hong Phuc Dang](https://github.com/hpdang), [Rafael Lee](https://github.com/RafaelLeeImg), AnnTran, Bella Phan.
- Project Repository:
  **[badgemagic-firmware](https://github.com/fossasia/badgemagic-firmware)**,
  [badgemagic-hardware](https://github.com/fossasia/badgemagic-hardware),
  [badgemagic-case](https://github.com/fossasia/badgemagic-case),
  [badgemagic-android](https://github.com/fossasia/badgemagic-android).
- Link to [GSoC site for this project](https://summerofcode.withgoogle.com/programs/2024/projects/KZ3lsPEG).

## Project Overview

LED name badges are small matrix LEDs that can be written text, and images, and
create animations over Bluetooth or USB connection. The Badge Magic app is an
open-source app used to send data to these LED badges. However, the app is still
limited and depends on these LED badge features. This project aims to implement
an open-source firmware for this type of LED Badge that uses a CH582
microcontroller, adds more features, as a base example for later hacks, and
merges it to the badgemagic-firmware repo of FOSSASIA.

### Goals

- Receiving bitmap data from the app over BLE.
- Receiving bitmap data over USB.
- Saving bitmap data to flash.
- Displaying bitmap on screen.
- Animations: scroll leftwards, scroll , scroll up, scroll down, fixed, "animation", "snowflake", "picture", "laser".
- Buttons: turning on, turning off, entering Bluetooth download mode, adjusting brightness.
- Firmware Optimizing for low-power.
- Repurposing the LED Badge to an audio visualizer over Bluetooth.
- Drawing a schematic and creating a BOM list for referencing.
- Documentation for the firmware.

### Accomplishments

The firmware now is capable of:

- Receiving bitmap data from the app over BLE.
- Having two 2 USB Composite Devices: HID and CDC-ACM.
- Receiving bitmap data over USB HID and CDC-ACM.
- Saving bitmap data to flash.
- Displaying bitmap on screen.
- Animations: scroll leftwards, scroll , scroll up, scroll down, fixed, "animation", "snowflake", "picture", "laser".
- Additional animations for hardcoded bitmap: scroll up, scroll down, infinite scroll.
- Animation and display version on charging state.
- Buttons: turning on, turning off, entering Bluetooth download mode, adjusting brightness.

I designed a open source hardware and a case along the way:

- Improved its design, made a PCB.
- Added a microphone to turn the badge to an audio visualizer.
- Wired the UART logging port to USB-C for a handy debugging instead of opening the case.
- Made a 3D case design.

Documentation for the firmware.

### What's left

I've exceeded designing a open source hardware and a case along the way. But this cost more time and so was not completed the audio visualizer.

## Contributions

### Community Bonding Period

- BadgeBLE: add description of uploading behavior - [badgemagic-firmware#7](https://github.com/fossasia/badgemagic-firmware/pull/7)
- Fix deployment pipeline to apk branch - [badgemagic-android#902](https://github.com/fossasia/badgemagic-android/pull/902)
- chore: fix build errors caused by lint check - [badgemagic-android#903](https://github.com/fossasia/badgemagic-android/pull/903)
- fix: Crash caused by missing permission - [badgemagic-android#910](https://github.com/fossasia/badgemagic-android/pull/910)
- fix: Crash due to old version of klock - [badgemagic-android#911](https://github.com/fossasia/badgemagic-android/pull/911)
- Fix warning flood - [badgemagic-android#913](https://github.com/fossasia/badgemagic-android/pull/913)
- fix android build check failed on PRs - [badgemagic-android#914](https://github.com/fossasia/badgemagic-android/pull/914)
- chore: remove ios CI check - [badgemagic-android#919](https://github.com/fossasia/badgemagic-android/pull/919)
- fix: Crash caused by missing permission on Android 12 - [badgemagic-android#932](https://github.com/fossasia/badgemagic-android/pull/932)
- fix: update prominent disclosures - [badgemagic-android#934](https://github.com/fossasia/badgemagic-android/pull/934)
- fix: remove completely location permissions - [badgemagic-android#936](https://github.com/fossasia/badgemagic-android/pull/936)
- Add KiCad schematics - [badgemagic-hardware#8](https://github.com/fossasia/badgemagic-hardware/pull/8)

### Coding Period

#### Phase 1

- Add Platform, Parts list, Hardware specs to README.md - [badgemagic-hardware#9](https://github.com/fossasia/badgemagic-hardware/pull/9)
- fix: wrong connection - [badgemagic-hardware#10](https://github.com/fossasia/badgemagic-hardware/pull/10)
- Add basic Charlieplexing scan - [badgemagic-firmware#10](https://github.com/fossasia/badgemagic-firmware/pull/10)
- Add basic button functionalities - [badgemagic-firmware#11](https://github.com/fossasia/badgemagic-firmware/pull/11)
- Add brightness adjustment - [badgemagic-firmware#12](https://github.com/fossasia/badgemagic-firmware/pull/12)
- Add support for .xbm bitmap file - [badgemagic-firmware#13](https://github.com/fossasia/badgemagic-firmware/pull/13)
- Add dynamic framebuffer - [badgemagic-firmware#14](https://github.com/fossasia/badgemagic-firmware/pull/14)
- Add animations - [badgemagic-firmware#15](https://github.com/fossasia/badgemagic-firmware/pull/15)
- Add entering power-off mode - [badgemagic-firmware#16](https://github.com/fossasia/badgemagic-firmware/pull/16)
- Add bluetooth animation in ble download mode - [badgemagic-firmware#17](https://github.com/fossasia/badgemagic-firmware/pull/17)
- fix: first leds of the first two columns are switched - [badgemagic-firmware#19](https://github.com/fossasia/badgemagic-firmware/pull/19)
- Add build pipeline - [badgemagic-firmware#20](https://github.com/fossasia/badgemagic-firmware/pull/20)
- fix: missing deallocating - [badgemagic-firmware#21](https://github.com/fossasia/badgemagic-firmware/pull/21)
- fix: Framebuffer linked list is misplaced on append - [badgemagic-firmware#28](https://github.com/fossasia/badgemagic-firmware/pull/28)
- fix: Out of bounds on framebuffer access - [badgemagic-firmware#29](https://github.com/fossasia/badgemagic-firmware/pull/29)
- feat: Bluetooth LE initial setup - [badgemagic-firmware#30](https://github.com/fossasia/badgemagic-firmware/pull/30)
- feat: Save received data from BLE to flash - [badgemagic-firmware#31](https://github.com/fossasia/badgemagic-firmware/pull/31)
- feat: Receive data over USB - [badgemagic-firmware#35](https://github.com/fossasia/badgemagic-firmware/pull/35)

#### Phase 2

- fix: schematic regulator's part number - [badgemagic-hardware#12](https://github.com/fossasia/badgemagic-hardware/pull/12)
- Badge Magic PCB layout - [badgemagic-hardware#14](https://github.com/fossasia/badgemagic-hardware/pull/14)
- fix: incompatible dimensions and unconnected leds - [badgemagic-hardware#17](https://github.com/fossasia/badgemagic-hardware/pull/17)
- fix: crash loop on new chip - [badgemagic-firmware#45](https://github.com/fossasia/badgemagic-firmware/pull/45)
- chore: Add release drafter - [badgemagic-firmware#47](https://github.com/fossasia/badgemagic-firmware/pull/47)
- feat: play all available bitmap slots sequentially - [badgemagic-firmware#49](https://github.com/fossasia/badgemagic-firmware/pull/49)
- feat: Charging status - [badgemagic-firmware#50](https://github.com/fossasia/badgemagic-firmware/pull/50)
- Prototype case - [badgemagic-case#2](https://github.com/fossasia/badgemagic-case/pull/2)
- feat: Add a build option for USB-C version - [badgemagic-firmware#55](https://github.com/fossasia/badgemagic-firmware/pull/55)
- feat: Automatically embedding version numbers - [badgemagic-firmware#53](https://github.com/fossasia/badgemagic-firmware/pull/53)
- chore: Update README for Installation, Usage and Development - [badgemagic-firmware#58](https://github.com/fossasia/badgemagic-firmware/pull/58)

#### Other Contributions

- fix: BLE won't work on Android 11 and below [badgemagic-android#968](https://github.com/fossasia/badgemagic-android/pull/968)
- fix: pipeline broken on PR [badgemagic-android#970](https://github.com/fossasia/badgemagic-android/pull/970)
- chore: separated pipeline for pull request [badgemagic-android#971](https://github.com/fossasia/badgemagic-android/pull/971)

## Conclusion

### What did I learn

Bare-metal USB Device Implementation:
At the moment, `tinyusb` doesn't support the CH582 chip. I implemented the USB device from the ground up without using any framework, which helped me better understand the USB protocol layer. Implementing a USB device on a bare-metal USB peripheral has always been on my learning list.

USB Device Class Implementation:
I had basic experience working with USB HID devices using Obdev V-USB, but only manipulating data at the higher level of the HID layer. Throughout this project, I implemented both a USB HID and a USB CDC-ACM. This required studying the specifications to configure and understand the topology of each class.

BLE GATT and ATT Layer:
Before this project, I had worked with BLE in a couple of projects but at a higher abstraction level. However, this project’s bare-metal BLE implementation allowed me to deepen my understanding of the GATT and ATT layers.

Git Force Push:
Before joining GSoC for this project, I studied several projects, but they're using mailing lists. For patches that needed improvements, contributors would resend updated versions to the list. I didn't know how GitHub’s PR workflow worked until my mentor told me to use `git rebase -i` and force push.

Communication:
Before GSoC, I wasn't familiar with how to contribute to a GitHub-hosted project, and I was initially hesitant to do so.

Collaboration:
I learned how to properly use Scrum. The weekly meetings were highly effective in setting clear goals for the next steps.

### Future Work

The work that left: Repurpose the LED Badge to an audio visualizer.

The ideas that came up, while doing this project, some of them were posted on the repo's issue:

- Clock feature [badgemagic-firmware#36](https://github.com/fossasia/badgemagic-firmware/issues/36)
- Bitmap streaming [badgemagic-firmware#41](https://github.com/fossasia/badgemagic-firmware/issues/41)
- Turning the badge on/off remotely [badgemagic-firmware#32](https://github.com/fossasia/badgemagic-firmware/issues/32)
- Alway-on BLE [badgemagic-firmware#34](https://github.com/fossasia/badgemagic-firmware/issues/34)
- Flash firmware over BLE [badgemagic-firmware#37](https://github.com/fossasia/badgemagic-firmware/issues/37)
- Option to disable microphone [badgemagic-hardware#18](https://github.com/fossasia/badgemagic-hardware/issues/18)

The piece of hardware wating to get used: Using microphone on the [badgemagic-hardware](https://github.com/fossasia/badgemagic-hardware) to implement the audio spectrum visualizer.

## Images

![FOSSASIA Badge Magic LED's charging animation](/assets/fossasia-badge-magic-open-firmware-google-summer-of-code-2024/charging-animation.gif)
_Charging animation_

![FOSSASIA Badge Magic Hardware: First PCB test batch](/assets/fossasia-badge-magic-open-firmware-google-summer-of-code-2024/badgemagic-hardware-pcb-first-batch.jpeg)
_Badge Magic Hardware: first test batch_

![FOSSASIA Badge Magic Hardware: Second PCB test batch](/assets/fossasia-badge-magic-open-firmware-google-summer-of-code-2024/badgemagic-hardware-pcb-2nd-batch.jpeg)
_Badge Magic Hardware: second test batch_

## Other Relevant Links

- [Scrums](https://groups.google.com/g/badgemagic).
- Issues:
  - [badgemagic-firmware](https://github.com/fossasia/badgemagic-firmware/issues?q=is%3Aissue+author%3Akienvo),
  - [badgemagic-hardware](https://github.com/fossasia/badgemagic-hardware/issues?q=is%3Aissue+author%3Akienvo),
  - [badgemagic-case](https://github.com/fossasia/badgemagic-case/issues?q=is%3Aissue+author%3Akienvo),
  - [badgemagic-android](https://github.com/fossasia/badgemagic-android/issues?q=is%3Aissue+author%3Akienvo).
- Pull Requests:
  - [badgemagic-firmware](https://github.com/fossasia/badgemagic-firmware/issues?q=is%3Apr+author%3Akienvo),
  - [badgemagic-hardware](https://github.com/fossasia/badgemagic-hardware/issues?q=is%3Apr+author%3Akienvo),
  - [badgemagic-case](https://github.com/fossasia/badgemagic-case/issues?q=is%3Apr+author%3Akienvo),
  - [badgemagic-android](https://github.com/fossasia/badgemagic-android/issues?q=is%3Apr+author%3Akienvo).

<!-- ## Acknowledgments
add later
 -->
---
layout: post
title:  "Cross compiling kernel modules"
image: ''
date:   2021-11-18 23:27
tags: [kernel, arm, modules, driver, cross-compile]
description: 'Note for config linux kernel build'
categories: [linux]
---

Just got to compile a driver for `ti-ads1220` running with `Orange Pi Zero`. 
I've figure out some stuff when digging to kernel modules. 
Below is how I config cross build for my Pi.

## 1. Target system

`Orange Pi Zero` running `Armbian Bionic 20.02.1` with `Linux 5.4.20-sunxi`

![alt target-system](/assets/cross-compiling-kernel-modules/target-system.png "armbian orange pi zero")

## 2. Download kernel source

Download kernel source from [kernel.org](https://mirrors.edge.kernel.org/pub/linux/kernel/)

The version of kernel source need to be download, must be exact version of 
the target system. Otherwise, you would be end up with:

![alt mod_unload](/assets/cross-compiling-kernel-modules/mod_unload.png "mod_unload caused by different magic version")

My target is `linux-5.4.20-sunxi` so I downloaded `linux-5.4.20.tar.gz`. 
Note that the `-sunxi` suffix could be add later by `.config` file.

## 3. Cross compile  toolchain

`arm-linux-gnueabihf` working well for me. 
On Debian just run:
```sh
$ sudo apt install gcc-arm-linux-gnueabihf
``` 
and every things works like a charm. Google it for more information or how to install it on other distro.

## 4. The kernel build config

The build configuration must be exact the same configuration 
of our target system.
You can get a copy from a running system:
- /proc/config.gz (the file is zipped)
- /boot/config
- /boot/config-*

In my case, I'm using:
```sh
~$ zcat /proc/config > .config
```

then use `scp` to copy `.config` from Pi to **root folder of kernel source** has just
downloaded.

## 5. Linux kernel suffix

### TL;DR

My target is `linux-5.4.20-sunxi` but the source is `linux-5.4.20` when I got 
my first-complete-module-build-without-error output file, I was happy to 
test it on my Pi. But unfortunately, I got `Invalid module format` when using
insmod to load my module. It took me 2 days to figure out what I'm doing wrong.

Things is the version magic of my module should match every single letter to 
the target version. For example, in my case, `vermagic` of my module was:
```
linux-5.4.20 SMP mod_unload ARMv7 thumb2 p2v8
```
while my Pi is:
```
linux-5.4.20-sunxi SMP mod_unload ARMv7 thumb2 p2v8
```
You can use `modinfo` for checking `vermagic` of module and `uname -a` for 
a running target.

### Fix it

The `.config` file you had taken from the running target, 
change `CONFIG_LOCALVERSION=""` to `CONFIG_LOCALVERSION="-sunxi"` before run 
make and everything should work properly.

## 6. Build the kernel

From **root folder of kernel source** run:
```sh
# update current .config file
$ make ARCH=arm CROSS_COMPILE=<arm-linux-gnueabihf- oldconfig
# build 
$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4
```
Once done, kernel directory are ready to build your module.

## 7. Build the module

A simple Makefile

```Makefile
obj-m += mymodule.o

KDIR=<your/path/to/kernel/source/root/folder>
CROSS=<toolchain compiler>

all:
	make ARCH=arm CROSS_COMPILE=$(CROSS) -C $(KDIR) M=$(PWD) modules
clean:
	make ARCH=arm CROSS_COMPILE=$(CROSS) -C $(KDIR) M=$(PWD) clean
```

From your module source code, run:

```sh 
$ make
```

<!-- just for trigger deploy -->
































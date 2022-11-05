---
layout: post
title:  "Windows Qt application deployment with static linking"
image: '/assets/rpi3b+-undervoltage-uart-console/console.png'
date: 2022-11-04 12:00 +0700
tags: [qt, msys2, windows]
description: 'How to deploy a Qt application with on Windows using MSYS2.'
categories: [raspberry-pi]
---

## The problem

Deployment might be an unnecessary activity when developing softwares. It been a long time since I started to learn programming, I only make for myself softwares which would be abandon for years later. If you want to run it again, just recompile that code.

{: .prompt-warning}
> **But what if you want that software to run on your friend computer ?**
> 
> Did you ever copy your executable file to another Windows machine, and it cannot run, but still run on your machine perfectly? Want to install a whole toolchain, copy the code to build on that machine (or 1000 may be) ? But I know someone really do this, Install Visual Studio 2019 on another machine in order to run a C# Winform app he build =]]. 


There a two way to deploy an application to another Windows machine:

- Dynamic linking - copy numberous `.dll` files to executable file. Many huge software such as Adobe Photoshop, Google Chrome, etc is using this way. Because shared library alway fast and optimized. But, the size and number of files is large, this would be zipped and ship with a `installer`.
- Static linking - "One file to run them all". All the used library will be linked into a single executable file. But the file would get bigger and quite slow, not to mention compiling and linking speed, terribly slow.

{: .prompt-info}

> **Static linking would be easy for low-tech person to use the software, especially when the software is small.**
> 
> I have build a small [physic engine](https://github.com/kienvo/physics-engine-from-scratch) for learning purpose, my co-worker find it interesting. He want me to send him this software for his child to play. I didn't have time until now. This piece of software wasn't used Qt so this easy to link statically.
>
> One of my friends want me to build a motor conntrol pannel application that run on Windows for him. Once again I need to deploy this for him, since the last time I try to statically link [my app](https://github.com/kienvo/p10-frame-maker) was not succeed untill now.
> 

> I'm using Windows as a build machine, at all the time I use Ubuntu.

## MSYS2

`msys2` is the only tool that support Qt library, and can install additional library, and natively run  on Windows as I consider. You can take a look at [msys2 packages], it seem like almost every open source library which could installed on Linux, were compiled for msys2 and supported cross platform too. I'm doing the development on Linux then using Windows and msys2 to build the releases.

{: .prompt-danger}
> Sorry, I didn't use Qt Creator.

MSYS2 have `mingw-w64-x86_64-qt5-base` contain the base of Qt5 library. It have multiple extra package for multiple purpose. Especially `mingw-w64-x86_64-qt5-static`, contain static library and utility for linking.

For a normal build, you need 4 packages

- `msys-base` 
- `make`
- `mingw-w64-x86_64-gcc`
- `mingw-w64-x86_64-qt5`

And for statically linking, one additonal package

- `mingw-w64-x86_64-qt5-static`

{: .prompt-tip}
> `msys2` uses `pacman`, so using it is similar to Arch-based distros pacman
>
> - Install package: `pacman -Sy <package-name>`
> - List installed packages `pacman -Qe`

## Statically linking Qt

As default, `qmake` will perform a normal build. To specify a static build you need 2 things to done:

- Add `CONFIG += -static` to qmake's `.pro` file.
- Temporary add qt5-static directory to PATH environment with highest priority: `PATH=/mingw64/qt5-static/bin:$PATH`

You can confirm that qmake generated a statically-build Makefile when run make:

```bash
nhung@DESKTOP-7ESDEUJ MINGW64 /g/doan-thaibinh
$ echo $PATH # This is my configured PATH
/mingw64/qt5-static/bin:/mingw64/bin:/usr/local/bin:/usr/bin:/bin:/c/Windows/System32:/c/Windows:/c/Windows/System32/Wbem:/c/Windows/System32/WindowsPowerShell/v1.0/:/usr/bin/site_perl:/usr/bin/vendor_perl:/usr/bin/core_perl

nhung@DESKTOP-7ESDEUJ MINGW64 /g/doan-thaibinh
$ make # An example when make
make -f Makefile.Release
make[1]: Entering directory '/g/doan-thaibinh'
g++ -c -fno-keep-inline-dllexport -O2 -std=gnu++11 -Wall -Wextra -Wextra -ffunction-sections -fdata-sections -fexceptions -mthreads -DUNICODE -D_UNICODE -DWIN32 -DMINGW_HAS_SECURE_API=1 -DQT_NO_DEBUG -DQT_CHARTS_LIB -DQT_WIDGETS_LIB -DQT_QUICK_LIB -DQT_GUI_LIB -DQT_QMLMODELS_LIB -DQT_QML_LIB -DQT_NETWORK_LIB -DQT_CORE_LIB -DQT_NEEDS_QMAIN -I. -IC:/msys64/mingw64/qt5-static/include -IC:/msys64/mingw64/qt5-static/include/QtCharts -IC:/msys64/mingw64/qt5-static/include/QtWidgets -IC:/msys64/mingw64/qt5-static/include/QtQuick -IC:/msys64/mingw64/qt5-static/include/QtGui -IC:/msys64/mingw64/qt5-static/include/QtQmlModels -IC:/msys64/mingw64/qt5-static/include/QtQml -IC:/msys64/mingw64/qt5-static/include/QtNetwork -IC:/msys64/mingw64/qt5-static/include/QtCore -Ibuild/moc -I/include -IC:/msys64/mingw64/qt5-static/share/qt5/mkspecs/win32-g++  -o build/obj/main.o main.cpp
... # Continued

```

By now, the executable file could run on any Windows machine with a bit slower than dynamic executable file.

---
layout: post
title:  "Windows Qt application deployment with static linking"
date: 2022-11-04 12:00 +0700
tags: [qt, msys2, windows]
description: 'How to deploy a Qt application with on Windows using MSYS2.'
categories: [raspberry-pi]
---

## The problem

Did you ever copy your .exe file compiled from C on your machine to another
Windows machine, and it cannot run? Maybe you could resolve this problem by
installing the whole toolchain and copying the code to that machine. But that is
inconvenient. In this post, I will guide you to solve this problem with best
practices.

There a two ways to deploy an application written in C/C++ to another Windows machine:

- Dynamic linking - copy numerous .dll files pack with executable files. Many
  huge software such as Adobe Photoshop, Google Chrome, etc are used this way.
  Because shared library always fast and optimized. But, the size and number of
  files are large, this would be packed and shipped with an installer.
- Static linking - "One file to run them all". All the necessary libraries will
  be packed into a single executable file by the linker. Therefore, the file would
  get bigger and quite slow because it has to load a huge static executable
  file into memory.

## MSYS2

`msys2` is convenient when compile existed projects with `qmake` on Windows. It
can install additional libraries just like package managers on Linux. `msys2`
libraries were built to run natively on Windows. You can take a look at [msys2
packages], it seems had almost every open-source libraries, were compiled for
msys2 and supported cross-platform. I'm doing the development on Linux then
using Windows and msys2 to build and test for releases. 

There were others. Like Cygwin, wsl2,... But they are not native. And sorry, I
don't use Qt Creator. It overcomplicated things. MSYS2 has
`mingw-w64-x86_64-qt5-base` containing the base of Qt5 library. It has multiple
extra packages for multiple purposes. Especially `mingw-w64-x86_64-qt5-static`,
which contains a static library and utility for linking.

For a normal build, you need 4 packages

- `msys-base` 
- `make`
- `mingw-w64-x86_64-gcc`
- `mingw-w64-x86_64-qt5`

For statically linking, one additional package

- `mingw-w64-x86_64-qt5-static`

> `msys2` uses `pacman`, so using it is similar to Arch-based distros pacman
>
> - Install package: `pacman -Sy <package-name>`
> - List installed packages `pacman -Qe`

## Statically linking Qt

As default, `qmake` will perform a normal build. To specify a static build you need 2 things to be done:

- Add `CONFIG += -static` to qmake's `.pro` file.
- Temporary add `qt5-static` directory to PATH environment with the highest priority: `PATH=/mingw64/qt5-static/bin:$PATH`

You can confirm that qmake generated a statically-build Makefile when running make:

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

By now, the executable file could run on any Windows machine a bit slower than
the dynamic executable file. But, it was linked into one convenient simple file.
You can now send this file to your friend!

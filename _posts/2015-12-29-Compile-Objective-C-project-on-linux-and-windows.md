---
layout:     post
title:      "Compile Objective-C project on linux and windows"
subtitle:   "Compile Objective-C project (unar and lsar) on linux and windows"
date:       2015-12-29
author:     "Ryan"
header-img: "img/post-bg-2015.jpg"
tags:
    - Programming
    - linux
---

# Introduction

The Unarchiver is a Mac OS X app, but the underlying unarchiving code can run on other OSes, too. There is not yet any GUI app for any other OS, but there are two command-line tools, unar and lsar, which can be used on Mac OS X, Windows and Linux.

The advantages of unar and lsar:

* Support almost all normal format(rar, 7z, zip, tar, gz, bz2, xz, lzma, and so on)
* Support to uncompress and list the broken package, such as a zip or rar pakage transfered a part of the totall one.

Official Web Page: http://unarchiver.c3.cx

# Compile on linux

#### Ubuntu

Do the same as the official illustrate.

```sh
sudo apt-get install build-essential libgnustep-base-dev libz-dev libbz2-dev libssl-dev libicu-dev
cd XADMaster
make -f Makefile.linux
```

#### CentOS

```sh
sudo yum install make gcc gcc-c++ gnustep-base-devel openssl-devel libicu-devel
sudo yum install gcc-objc gcc-objc++ libobjc
cd XADMaster
make -f Makefile.linux
```
Make sure install gcc Objective-C compile support, or it will occure "gcc: error trying to exec 'cc1obj': execvp: No such file or directory".

# Compile on windows

#### Objective C Programming in Windows using GNUStep

GNUstep is a mature Framework, suited both for advanced GUI desktop applications as well as server applications. The framework closely follows Apple's Cocoa (formerly NeXT's OpenStep) APIs but is portable to a variety of platforms and architectures.

GNUstep offers Development tools for command-line and GUI development, as well as the foundations for a Desktop environment, which other projects can complete.

GNUStep Official Web Page: http://gnustep.org/

#### Install GNUStep and use it

Windows installer url: http://gnustep.org/windows/installer.html

Download and install GNUStep MSYS System, GNUStep Core and GNUStep Devel.

After that, invoke the shell (you will find it in start menu).

Find out where your home directory is. (Mine is "c:/GNUStep/msys/1.0/home/Administrator")

Copy the source code to home directory, then execuate compile command.

```sh
cd XADMaster
make -f Makefile.windows
```
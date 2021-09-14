---
layout: post
title: Getting the Nano to work
date: '2013-12-06T15:39:00+08:00'
tags:
- osx
- ftdi
- nano
- linux
tumblr_url: https://robokitchen.tumblr.com/post/69174503685/getting-the-nano-to-work
---
As discussed in a [previous post](http://robokitchen.tumblr.com/post/66812598033/uninstalling-the-ftdi-drivers) the FTDI driver required to communicate with the Nano is unusable on OSX. It regularly crashes the whole computer. Something in the toolchain is also unstable on OSX: I had several reboots when building. Some of them happened while no Arduino was even plugged into the USB.

I tried building on Windows XP with Parallels Desktop, but the lack of support for symbolic links is a deal breaker: I cannot compile my sketches.

Remains Linux. I had a VirtualBox virtual machine already set up, some version of xubuntu. I downloaded and installed the Arduino software, added my user to the dialout group, and after logging off and back on I could upload the blink test sketch to the Nano. Victory? Not so sure.

The Uno does not work under Linux, there’s something wrong in the upload stage. I have spent far too much time in an unsuccessful attempt to resolve the problem. Nevermind, at least the Nano works, I might even stop using the Uno altogether.


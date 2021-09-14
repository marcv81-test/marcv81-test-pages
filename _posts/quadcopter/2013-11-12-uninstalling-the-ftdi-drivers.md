---
layout: post
title: Uninstalling the FTDI drivers
date: '2013-11-12T22:20:00+08:00'
tags:
- arduino
- osx
- ftdi
---
I've had an Arduino Uno for some time now. The serial port worked out of the box in OSX 10.8, it didn't require any driver. When I got an Arduino Nano the serial port was not recognised: I had to install the [FTDI drivers](https://www.ftdichip.com/Drivers/VCP.htm). The version at the time was 2.2.18.

Immediately after that I started to notice that OSX crashed fairly often when an Arduino was plugged, maybe once every hour. Each time the computer froze for a few seconds before rebooting altogether. I wanted to uninstall the FTDI drivers to test if they were the source of the problem but there was no easy way to do it. I left my computer like that for a few days but eventually the problem became too annoying.

We can uninstall most OSX packages using the command line. First we list the installed files from the package receipt.

_lsbom -fls /Library/Receipts/FTDIUSBSerialDriver.pkg/Contents/Resources/FTDIUSBSerialDriver.bom_

We can then delete all these files.

_sudo rm -rf /System/Library/Extensions/FTDIUSBSerialDriver.kext_

And the package receipt too.

_sudo rm -rf /Library/Receipts/FTDIUSBSerialDriver.pkg_

To check that the driver was uninstalled I connected the Arduino Nano and verified that its serial port didn't appear in the Arduino IDE. The Arduino Uno remained visible as expected. It's now been a few days and OSX seems as stable as before. Clearly there's a problem with the FTDI driver. The current version was released more than a year ago so I'm not expecting a fix anytime soon. We'll have to find another OS to program the Arduino Nano.

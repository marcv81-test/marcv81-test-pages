---
layout: post
title: PS2 controller interface - Part 1
date: '2014-11-05T23:37:00+08:00'
tags:
- arduino
- ps2
- controller
- spi
- pullup
---
A few [articles](https://www.billporter.info/2010/06/05/playstation-2-controller-arduino-library-v1-0/) and [libraries](https://playground.arduino.cc/Main/AnalogPSXLibrary) explain how to interface an Arduino with a PS2 controller. None of them worked for me because they didn't explain how to connect the gamepad to the microcontroller, but I didn't realise that so I started a [new implementation](https://github.com/marcv81/quadcopter/tree/021c74899a0ac0c289ffa5c149bde3a148230bc4/libraries/PS2Controller) from scratch.

[Curious Inventor](https://store.curiousinventor.com/guides/PS2/) was my main source of information.

**Technical decisions**

The PS2 controller uses a variant of the the [SPI protocol](https://en.wikipedia.org/wiki/Serial_Peripheral_Interface_Bus). We can either bitbang the protocol or use the AVR hardware implementation.

Bitbang SPI pros:

- Can use any pins

Hardware SPI pros:

- Fast
- Saves program memory
- Known to work reliably

The only acceptable excuse to bitbang the protocol is if you need to use some of the hardware SPI pins for something else (e.g: PWM). I chose to rely on the AVR hardware implementation.

**Connection**

The PS2 controllers are reported to work around 3.3V when connected to the console. This is obviously the preferred supply voltage. However both my original and aftermarket controllers seem to work fine at 5V so I didn't bother with a level converter.

The SPI protocol requires 4 pins. On the Arduino Uno/Nano the SPI hardware pins are as follows.

- SS: any pin will do, pin 10 is generally used
- MOSI: pin 11
- MISO: pin 12
- CLK: pin 13

MISO needs a pull-up resistor. The integrated Arduino pull-up is said to be more than 20k which is way too high for this application. I used a 1k external resistor. The pull-up resistor was the reason I had no success with other people's libraries, but I only realised too late.

The PS2 controller SPI wiring is as follows.

- Command = MOSI
- Data = MISO
- Acknowledge = SS
- Clock = SCLK

The PS2 provides an extra signal wire (acknowledge) which is not part of the SPI protocol. We can ignore it.

**Low-level communication**

The low-level communication protocol is as follows.

1. Set SS low
2. Wait for 10us (determined empirically)
3. Send/receive a 8-bit word using SPI mode 3 LSB first at 500kHz
4. If there are more words to send/receive go to 2
5. Set SS high
6. Wait for 14us since setting SS high (or for 21us since the end of the last word) before sending another message (determined empirically)

During 6 we do not have to actively wait, we can do whatever else as long as it takes long enough.

The waiting times were determined empirically with a genuine controller. My aftermarket one tolerates slightly shorter delays. The waiting times were shortened until the point where reducing them any further would result in an unreliable connection. The exact delays were then measured using a logic analyser.

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2014-11-05-ps2-controller-interface-part-1-1.jpg)
{:refdef}

The above captures illustrates a real message with the channels in the following order.

1. SCLK
2. MISO
3. MOSI
4. SS

**High-level communication**

The low-level protocol allows us to send arbitrary messages made of several 8-bit words. The high-level protocol defines which messages are valid and their meaning. I only implemented one command (0x42). It does everything I require for now. The message is as follows.

1. Transmit 0x01, receive 0xFF
2. Transmit 0x42, receive 0x73 in analog mode or 0x41 in standard mode
3. Transmit 0x00, receive 0x5A
4. If some of the above data is invalid terminate the message now
5. If in analog mode read 6 words, if in standard mode only read 2 words

During 5 we send the word 0x00 enough times to read the required number of words.

In standard mode the 2 words contain the buttons status. In analog mode the first 2 words serve the same purpose, and the next 4 words contain the analog joystick values.

On average the implementation reads the analog mode status in 248us, or the standard mode status in 144us.

**Genuine vs. aftermarket controller**

This is a comparison with only one make of aftermarket controllers. There are other ones and you might be more lucky than me.

- The aftermarket controller is more tolerant to shorter transmission delays
- The original controller has 8-bit joysticks with values evenly spread across the entire stick movement range. The aftermarket controller has very poor joysticks: the values does not cover the full range of stick movement, and there are only 32 counted possible values instead of 255. I would avoid aftermarket controllers for that reason.

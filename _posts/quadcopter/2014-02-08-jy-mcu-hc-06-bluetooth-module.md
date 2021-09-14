---
layout: post
title: JY-MCU/HC-06 Bluetooth module
date: '2014-02-08T23:41:00+08:00'
tags:
- bluetooth
- jy-mcu
- hc-06
- arduino
- serial
---
I got a JY-MCU/HC-06 Bluetooth module on eBay for £5.89. That's cheap for a Bluetooth serial port, and it could come in handy. My unit reads JY-MCU v1.02 pro, and the HC-06 came pre-soldered. The soldering might require more precision than I'm capable of, so that's great.

**Connection**

The labels read: "Power: 3.6-6V" and "Level: 3.3V". The Internet says it works well on 5V but why risk it? We're going to step everything down to 3.3V (yes, even VCC, I don't trust labels).

- The Arduino Nano provides a 3.3V supply for VCC.
- With 2 resistors we can step the Arduino TX signal down. I had 2.2k ohm and 1k ohm handy, which we can bridge to step 5V down to 3.4V. Close enough.
- The RX should not require a step-up: the Arduino reads any signal above 2.5V as a high-level, so we may use the 3.3V as it is.

This worked well on the breadboard so I made a tiny stripboard. The track under the 1k ohm resistor is cut.

<figure class="tmblr-full" data-orig-height="179" data-orig-width="350" data-orig-src="https://64.media.tumblr.com/9489ec9772d755f7d277ce7733ec01cb/tumblr_inline_n0ovypXJJj1snd83q.jpg"><img alt="image" src="https://64.media.tumblr.com/9489ec9772d755f7d277ce7733ec01cb/tumblr_inline_pk0xnjPISt1snd83q_540.jpg" data-orig-height="179" data-orig-width="350" data-orig-src="https://64.media.tumblr.com/9489ec9772d755f7d277ce7733ec01cb/tumblr_inline_n0ovypXJJj1snd83q.jpg"></figure>

**Note:** I measured the voltage at the bridge and got 2.8V instead of 3.4V. Eeew, it turns out that the 5V pins of the Arduino Nano barely provide 4V when powered from USB. I could reproduce that with both a genuine Arduino board (possibly fried from previous experiments) and a clone (possibly poor quality). I checked the PC USB ports voltage, no problem there. I'm not too sure what's going on but it shouldn't happen when we run off batteries and regulated 5V, so let's not worry about it.

**Configuration**

To configure the Bluetooth module we have to send it AT commands through the TTL serial. I don't have a USB to TTL serial adapter, so we're going to program the Arduino to relay the serial communications between the PC and the module. We'll use the hardware serial port to communicate with the PC, and a [software serial port](https://arduino.cc/en/Reference/SoftwareSerial) on the pins 2 (RX) and 3 (TX) to communicate with the module. The SoftwareSerial library is temperamental: it fails to receive data correctly at 115200 bauds, and it requires some extra delay to send data at 9600 bauds. Here is [the relay sketch code](https://github.com/marcv81/quadcopter/tree/26eb118fea638ae734de5a37117ecdc54c8d517a/sketches/SerialRelay).

After connecting the Bluetooth module to the software serial port we can send it AT commands from the Arduino serial monitor. We can rename the device with AT+NAMERobokitchen and change the Bluetooth PIN with AT+PIN0000. This works well and I'm able to pair with the device. By default the module runs at 9600 bauds. We can increase the speed to 115200 bauds with AT+BAUD8. After that the communication won't synchronise until we update the serial speed and upload the sketch again. As mentioned above the received data is invalid at 115200 bauds, but we can still send commands and return to 9600 bauds with AT+BAUD4 if required.

<figure class="tmblr-full" data-orig-height="327" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/9e5611f7c5e7bb66d44bfd421a8678f0/tumblr_inline_n0p9v5jXK81snd83q.jpg"><img alt="image" src="https://64.media.tumblr.com/246b4773732a6ae60e4a2e504e0b283b/tumblr_inline_pk0xnjPElU1snd83q_540.jpg" data-orig-height="327" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/9e5611f7c5e7bb66d44bfd421a8678f0/tumblr_inline_n0p9v5jXK81snd83q.jpg"></figure>

**Integration**

Now that the Bluetooth module is configured we can connect it to the Arduino hardware serial port on the pins RX0 and TX1. These pins are also used for the USB serial communications, and we can't use both USB and Bluetooth at the same time. We'll have to disconnect VCC from the Bluetooth module whenever we want to use the USB, for instance when uploading a sketch.

As a test we uploaded the receiver test sketch created a few months ago. This sketch uses the OrangeRX receiver and the PPM sum stripboard, which we connected. We paired with the Bluetooth device and accessed the serial port with "screen /dev/tty.Robokitchen-DevB 115200". The expected output appeared immediately, can't beat that for plug & play!

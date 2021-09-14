---
layout: post
title: Stripboard
date: '2014-01-06T00:32:00+08:00'
tags:
- arduino
- nano
- stripboard
- servo
- i2c
---
 ![](https://64.media.tumblr.com/4296fdff64c1e2d7a8fad95fb8b6f2b7/tumblr_myyetlq6uv1sjwnlxo1_1280.jpg)  

I made a stripboard to replace the breadboard in case I actually want to embed the Arduino Nano onto a project. The two servo outputs are connected to the pins 5 and 6. The four wires are for the I2C bus in the following order: 5V, GND, SCL, SDA. There is a separate power supply for the high-power (servo) and low-power (microcontroller, I2C) devices, but they can be connected together if required. Despite my ham-fisted soldering hand the closed loop gimbal sketch works okay.

**Bonus:** a few intermittent bugs (MPU-6050 resets, too many I2C errors) got resolved after I started using the stripboard. They were presumably due to loose breadboard connections.

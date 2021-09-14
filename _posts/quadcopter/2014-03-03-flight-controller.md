---
layout: post
title: Flight controller
date: '2014-03-03T22:45:00+08:00'
tags:
- quadcopter
- multiwii
- flight controller
- build
---
A little while ago I put together a [flight controller stripboard design](https://github.com/marcv81/quadcopter/tree/977318e9e0fab4c24864249126d6a733ea9596f1/hardware/FlightController). While it really should work, I couldn't bring myself to cut, drill and solder it.

<figure class="tmblr-full" data-orig-height="297" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/3200db39279aee432c51855747a692af/tumblr_inline_n1vqmz2gxX1snd83q.jpg"><img alt="image" src="https://64.media.tumblr.com/67583374dae605c243e5e233e494b18c/tumblr_inline_pg5mgybP7z1snd83q_540.jpg" data-orig-height="297" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/3200db39279aee432c51855747a692af/tumblr_inline_n1vqmz2gxX1snd83q.jpg"></figure>

However the I2C sensors we're using (MPU-6050, HMC5883L, BMP085) and the Arduino microcontroller are exactly the same as on the MultiWii Crius 2.x SE boards, so I ordered one. I could not find the board schematics online, so I tested the following.

- The MPU-6050 and BMP085 test scripts run without any change.
- The HMC5883L test script does not find the device, neither does an I2C bus scan. My best guess is a **defective** board, please shout if you know better.
- The - pin, servo -, UART GND, and I2C GND are all connected together (no surprise).
- The + pin, servo +, UART 5V, and I2C 5V are connected together.

I thought the 5V/+ pins might not all be connected together. This would help powering a tricopter tail servo with a different power source than the Arduino and the receiver. A servo might use too much power and reduce the supply voltage enough for the Arduino or the receiver to reset. That happened when testing the servo gimbal.

We're using ESCs with integrated UBECs: because all the MultiWii 5V/+ pins are connected together we shall cut the ESCs red wires. Otherwise all the buck-converters would be connected in parallel, which would be evil. I'll leave just one red wire to power the MultiWii board from that UBEC.

<figure class="tmblr-full" data-orig-height="334" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/562b953cba4cbb6b2e1b0c7a754f1811/tumblr_inline_n1vr7yhz4Z1snd83q.jpg"><img alt="image" src="https://64.media.tumblr.com/95a30c08d19a9f6788ed794fd841ff44/tumblr_inline_pg5mgzRY731snd83q_540.jpg" data-orig-height="334" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/562b953cba4cbb6b2e1b0c7a754f1811/tumblr_inline_n1vr7yhz4Z1snd83q.jpg"></figure>

I secured the flight controller on a plastic card and attached it to the [frame](https://robokitchen.tumblr.com/post/72602724137/diy-artf-quadcopter) with the [PPM sum stripboard](https://robokitchen.tumblr.com/post/65192427388/ppm-sum-stripboard-experiments) and its OrangeRX radio receiver. It was more practical to configure it as a +4 rather than a X4. The radio receiver (pin 3) is on the back arm, and the motors shall run as follows.

- Front: CW (pin 10)
- Left: CCW (pin 11)
- Back: CW (pin 5)
- Right: CCW (pin 6)

I put together a [test sketch](https://github.com/marcv81/quadcopter/tree/master/sketches-util/QuadMotorTest). It first arms the ESCs: the delays are empirical but seem to work well every time. Then the TX sticks are mapped to a different motor each. It would never fly like that, but it's good for testing that every motor works. Also the motors shut down when the TX is powered off, which is a good safety feature. Everything works well, and I rewired the motors to make sure they run in the right direction.

**EDIT:** The pins have changed since we are now using [hardware PWM](https://robokitchen.tumblr.com/post/84759084465/490hz-esc-control).

- Front: pin 3
- Left: pin 9
- Back: pin 10
- Right: pin 11

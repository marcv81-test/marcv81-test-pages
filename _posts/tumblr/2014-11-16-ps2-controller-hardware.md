---
layout: post
title: PS2 controller hardware
date: '2014-11-16T00:21:00+08:00'
tags:
- arduino
- pro mini
- ps2
- controller
- pull-up
- spi
tumblr_url: https://robokitchen.tumblr.com/post/102734615530/ps2-controller-hardware
---
So far we connected the PS2 controller to an Arduino Nano using a breadboard. This isn’t ideal to use in the field. We could create a shield, but instead we’re going to directly solder the PS2 controller extension lead to a Pro Mini.

Below are the soldering points.

- Pin 10 = SS (yellow)
- Pin 11 = MOSI (orange)
- Pin 12 = MISO (brown) + 1k pull-up
- Pin 13 = SCLK (blue)
- VCC = Power (red) + 1k pull-up
- GND = Ground (black) + battery ground
- RAW = Motors power (grey) + battery power

The clock pin cannot be accessed once the 1k pull-up is mounted, so we soldered it first. Also the battery wires were too large so we made a smaller gauge extension. I used a 5V Arduino Pro Mini, maybe I would have been better of with a 3.3V one.

<figure class="tmblr-full" data-orig-height="357" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/5e0336dc3fc79ebe88db74ec25538f1d/tumblr_inline_nf3vtdHn681snd83q.jpg"><img alt="image" src="https://64.media.tumblr.com/93c9477c8405ebf26cf70fcac31cfd42/tumblr_inline_pkpqk2BnvZ1snd83q_540.jpg" data-orig-height="357" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/5e0336dc3fc79ebe88db74ec25538f1d/tumblr_inline_nf3vtdHn681snd83q.jpg"></figure>

Despite some questionable soldering technique it all works fine with the [existing sketch](https://github.com/marcv81/robokitchen/tree/021c74899a0ac0c289ffa5c149bde3a148230bc4/sketches/PS2Test).


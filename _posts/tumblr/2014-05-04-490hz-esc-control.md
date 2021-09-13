---
layout: post
title: 490Hz ESC control
date: '2014-05-04T22:36:00+08:00'
tags:
- arduino
- timer
- pwm
- esc
- 490hz
tumblr_url: https://robokitchen.tumblr.com/post/84759084465/490hz-esc-control
---
Due to some potentially unrelated issues I researched how to [better implement the PWMs controlling the ESCs](https://github.com/marcv81/robokitchen/commit/1eb4d749f2f42326dd6c64bfe44bccbfa50e4422). I’m not sure it was absolutely required but now it’s there, it works, and for the moment I’m not reverting to see what would happen.

The new implementation supports a 490Hz refresh rate instead of 50Hz: this might help to better recover from an out-of-timing high-level pulse, but I wouldn’t expect any. We use hardware rather than software PWM: I can’t find my Saleae anymore, but I would expect a better timing accuracy. Also the timing resolution is now 8ms instead of 4ms: this could yield poorer results and I may revert to the old library to compare the results in the future.

The implementation is very straightforward. The [ATmega328p timers](http://www.atmel.com/Images/Atmel-8271-8-bit-AVR-Microcontroller-ATmega48A-48PA-88A-88PA-168A-168PA-328-328P_datasheet.pdf) are configured to the following values:

Timer 1 (pins 9 and 10):

- WGM1 = 0001: PWM, Phase Correct, 8-bit, top = 0xFF.
- CS1 = 011: Prescaler set to 64.
- COM1A = 10: Clear OC1A on Compare Match when up-counting. Set OC1A on Compare Match when down-counting.
- COM1B = 10: Clear OC1B on Compare Match when up-counting. Set OC1B on Compare Match when down-counting.

Timer 2 (pins 11 and 2):

- WGM2 = 001: PWM, Phase Correct, 8-bit, top = 0xFF.
- CS2 = 100: Prescaler set to 64.
- COM2A = 10: Clear OC2A on Compare Match when up-counting. Set OC2A on Compare Match when down-counting.
- COM2B = 10: Clear OC2B on Compare Match when up-counting. Set OC2B on Compare Match when down-counting.

The above initialises a 490Hz PWM on the pins 9, 10, 11, and 3. We control the duty cycles&nbsp;setting OCR1AL, OCR1BL, OCR2AL, or OCR2BL. The pins cannot be changed, they are hardwired to the timers. There is no need to program interrupts, once set the PWMs and duty cycles remain until the micro-controller is power cycled (danger in case of a software crash!). The [updated motors test sketch](https://github.com/marcv81/robokitchen/blob/1eb4d749f2f42326dd6c64bfe44bccbfa50e4422/sketches-util/QuadMotorTest/QuadMotorTest.ino) works without any problem.


---
layout: post
title: Driving servos
date: '2013-12-12T21:14:00+08:00'
tags:
- servo
- pwm
- timer
---
I ordered two [MG996R servos](https://www.servodatabase.com/servo/towerpro/mg996r) for £10.99 to experiment with. I wasn't satisfied with any of the libraries I tried so I decided to implement my own. We'll use interrupts to allow other operations to happen while driving the servos.

Most servos expect a pulse lasting between one and two milliseconds every 20 milliseconds (at 50Hz). We'll divide the 20ms frame duration into 8 channels lasting 2.5ms each. We'll reserve the last channel for synchronisation, so the library can drive up to 7 servos.

We'll define two classes:

- ServoTimer manages the timing and calls Servo when required.
- Servo implements a state machine to set the pins high and low and drive the servos.

**ServoTimer**

Timer durations shall be stored in the timer-friendly format. To wait for a specific duration the timer 2 is first programmed to tick at a fixed frequency a number of times, then reprogrammed so that the last tick happens exactly after the target duration. Our timer-friendly format shall contain a primary integer representing the number of fixed frequency ticks, and a secondary integer to reprogram for the last tick.

Durations shall not be modified while being used by the timer. To ensure this we'll copy the active duration to a cache before using it. We'll only modify the cache in the ISR, when the active duration has expired and we don't rely on it anymore.

Because we cache the active duration during an interrupt, and because a duration is larger than a byte, we also need to make sure the source duration is not being updated while we copy it. We'll hence disable the interrupts when modifying a duration which could be used as a source by the timer.

**Servo**

When the servo ISR is called it performs the required action to drive the servos and then resets the timer. This is all implemented using a state machine which is called whenever the timer expires.

Each servo cycle shall last exactly 2.5ms. For this we'll store two durations per channel: one to time the high level state, and the other to time the low level state and fill in the 2.5ms cycle.

<figure class="tmblr-full" data-orig-height="319" data-orig-width="441" data-orig-src="https://64.media.tumblr.com/b8a58bdfe8c244d4e99d0afc7b46a96e/tumblr_inline_mzd8a1v86I1snd83q.jpg"><img src="https://64.media.tumblr.com/b8a58bdfe8c244d4e99d0afc7b46a96e/tumblr_inline_pk37plh1j41snd83q_540.jpg" data-orig-height="319" data-orig-width="441" data-orig-src="https://64.media.tumblr.com/b8a58bdfe8c244d4e99d0afc7b46a96e/tumblr_inline_mzd8a1v86I1snd83q.jpg"></figure>

This was all implemented in [a library and a test sketch](https://github.com/marcv81/quadcopter/commit/20cf3c92964080e0a8e5f8eecdaf67d4c9e0fc57).

The test sketch rapidly sets the channel 0 to 1, 1.2, 1.4, 1.6, 1.8, or 2ms. The channel 1 is constantly set to 1.5ms. Because the ISR takes some time to run we need to subtract a delay from the different durations in order to reach the correct timings. The delays are determined experimentally based on the average measured error.

The results look good after adjusting the delays.

<figure class="tmblr-full" data-orig-height="225" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/0bf8e872cd57b085328c32ad657f8355/tumblr_inline_mxppbk44D51snd83q.jpg"><img alt="image" src="https://64.media.tumblr.com/1e891dc2163bf6bf11dc89b63c482c4d/tumblr_inline_pk37pmms3s1snd83q_540.jpg" data-orig-height="225" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/0bf8e872cd57b085328c32ad657f8355/tumblr_inline_mxppbk44D51snd83q.jpg"></figure>

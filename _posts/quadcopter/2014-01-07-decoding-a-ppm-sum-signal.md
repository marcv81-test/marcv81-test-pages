---
layout: post
title: Decoding a PPM sum signal
date: '2014-01-07T00:10:00+08:00'
tags:
- ppm
- decoder
- pwm
- arduino
---
We saw in a [previous post](https://robokitchen.tumblr.com/post/65192427388/ppm-sum-stripboard-experiments) how to convert a 6 channels OrangeRX receiver output to a PPM sum. Let's decode this signal.

The advantage of a PPM sum is that the signal only uses one pin. As discussed before, each value is transmitted as a 1-2ms high-level pulse during the 2.5ms allocated to the channel, a frame lasts 20ms, and the last channel is always left empty for synchronisation. So a PPM signal can contain a maximum of 7 channels.

We'll use pin interrupts to time the pulse of each channel precisely without blocking the rest of the sketch. Pin interrupts can only be set on the Arduino Nano pins 2 and 3. We'll use the latter. Different pin interrupt modes are available, we'll use CHANGE: when the logic level of the pin changes, the interrupt is triggered.

We'll define a frame structure to store the decoded data. The program uses three frames and swaps between them cyclically using indices.

- A first frame is currently being decoded from the PPM signal.
- A second frame was previously being decoded, and is now available for the rest of the sketch to read.
- A third frame acts as a buffer. When the frame being decoded finishes we update the indices to cycle through the frames. With only two frames, the available frame would become the currently being decoded frame: if the sketch had been reading the available frame, its contents would become corrupt.

Frames contain the following fields.

- An array to store the duration of each channel.
- The active channel. When the frame is being decoded it tells us where to store the next duration. When the frame is available it allows us to verify the total number of channels.
- The start time of the frame. It allows us to detect and discard outdated frames in case we can't decode new frames.
- Whether or not the frame has been completely decoded yet. This is only useful while decoding the very first frame, in order to ensure the previous one is not considered available.

The refresh() function parses the available frame all at once and stores the results in a user-friendly format for later use. The function checks the following.

- The available frame is not outdated.
- The available frame was marked as completely decoded.
- The available frame contains the expected number of channels.
- The duration of each channel is between 1ms and 2ms.

The channel durations are converted to signed bytes (between -128 and 127). There is no loss of precision because the timer 0 precision is $$4\mu s$$, and $$(2000-1000) \div 4 = 250$$, which fits nicely into 8 bits. The value 0x80 is reserved for read errors.

The interrupt service routine does not execute instantly and we could use constants to fix the timing, but we can just trim the transmitter instead.

The code is [here](https://github.com/marcv81/quadcopter/tree/a32614a492ecde217b623cacdd636e1becf3cfdd/sketches/RXTest). I added a servo input on the pin 3 to my [stripboard](https://robokitchen.tumblr.com/post/72379908472/i-made-a-stripboard-to-replace-the-breadboard-in), things work nicely.

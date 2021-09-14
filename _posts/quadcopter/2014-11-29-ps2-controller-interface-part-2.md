---
layout: post
title: PS2 controller interface - Part 2
date: '2014-11-29T22:08:49+08:00'
tags:
- arduino
- ps2
- controller
- hotplug
- rumble
- motors
- analog mode
- finite-state machine
tumblr_url: https://robokitchen.tumblr.com/post/103922066495/ps2-controller-interface-part-2
---
In the previous post we polled the PS2 controller for whichever data it would provide, but we can get more out of it. I’m particularly interested in the rumble motors: we could get feedback without having to look at LEDs or a screen.

The PS2 controller supports different commands. We previously relied on a single command: poll (0x42). To access the advanced features we have to send other commands. Curious Inventor [lists them all](http://store.curiousinventor.com/guides/PS2/).

I first tried to force the controller into analog mode as it looked easier than configuring the rumble motors: we have to send “enter config” (0x43), then “enable analog mode” (0x44), and finally “exit config” (0x43). I [significantly reworked the library](https://github.com/marcv81/robokitchen/commit/1a0810d78fed32c2528038b5513e752bd71f31b7) to support arbitrary commands. In the process I discovered that my earlier estimate for the delay between frames (15-20us) was seriously underestimated: 4ms seems to yield much better results. Shorter delays may result in a 3rd byte of the header response that is different from the expected 0x5a. I believe the other values I encountered (0x00, 0xff, 0xfe) might be error codes but I did not investigate further.

It started as a simple test but forcing the controller into analog mode is actually a nice feature. We don’t have to worry about the mode: we can just read the analog sticks whenever. Configuring the controller at the beginning of the sketch works well, but problems may arise if it gets disconnected/reconnected or if the mode changes for other unforeseen reasons. In order to address this I added a counter to [reconfigure the controller on a regular basis](https://github.com/marcv81/robokitchen/commit/31b18e2c6eceb71cf2fbb16e9f673c1e4555f4dd). I preferred this strategy to reconfiguring only if the mode was not analog: I didn’t want to make too many assumptions on the problems which might arise.

At this stage adding support for the rumble motors is trivial. We just need to call “enable rumble” (0x4d) during our configuration sequence and map the motors to the first 2 bytes transmitted to the controller during the poll command (0x42). This requires a minor change to the poll() function and adding 2 functions to control a motor each. I also [updated the sketch](https://github.com/marcv81/robokitchen/commit/152209d4ecc2a96ab6e4544efb5bd11087204637) so that the X and O buttons would make the gamepad vibrate.

The implementation uses a lot of delays, one of which blocks the microcontroller for 4ms at a time. This is inefficient and we may want to run other code while the gamepad is getting ready. I [divided the configuration sequence in different states](https://github.com/marcv81/robokitchen/commit/0cfce8f22805741d96f2f8d3c08c19aa2478f50f) and implemented them as a [finite-state machine](http://en.wikipedia.org/wiki/Finite-state_machine). Each call to update() checks that enough time has elapsed and runs the next command in the sequence according to the following flowchart.

<figure class="tmblr-full" data-orig-height="500" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/7517a2d3d235b3da5815ea5a9593f55d/tumblr_inline_nftl2mmIvz1snd83q.jpg"><img alt="image" src="https://64.media.tumblr.com/78964a1e76d0a9ab6e07221a7181cd0a/tumblr_inline_pcka91u7r61snd83q_540.jpg" data-orig-height="500" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/7517a2d3d235b3da5815ea5a9593f55d/tumblr_inline_nftl2mmIvz1snd83q.jpg"></figure>

We now have a PS2 controller library which supports hotplug and allows us to control the rumble motors. The update() function takes a maximum of 270us at a time, which allows us to run plenty of other code.


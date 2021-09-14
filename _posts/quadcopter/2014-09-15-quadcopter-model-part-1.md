---
layout: post
title: Lift force measurement
date: '2014-09-15T00:00:00+08:00'
tags:
- torque
- force
- rpm
- scale
- weight
- gravity
- simonk
- linear
- quadratic
- model
---
Let's measure how much lift force each motor/propeller is generating.

**Measure**  
  
The idea is to put the quadcopter on a scale and observe how much less weight is measured when a motor spins. To reduce the disturbances this should be done indoors and with enough clearance to avoid any ground effect. As all the motors/propellers should produce similar results I decided to measure the front one only.  
  
I [modified the QuadMotorTest sketch](https://github.com/marcv81/quadcopter/commit/71c3a7c3d8ce4d86e915c37da0a31218179a8c6f) to help us with the measure.

- The PWM high-level time is now printed on the serial port.
- The PWM is now adjustable on the AUX channel which I set to a knob on the transmitter.
- The elevator and ailerons now activate the motors independently, and the throttle and rudder activate the pairs of opposite motors.

It is now easy to control exactly the input of each ESC/motor. It will help us relate the PWM to the generated lift force.

<figure class="tmblr-full" data-orig-height="605" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/5f171945926c2ff6369d4b2f73b75e87/tumblr_inline_nbwm74ZbD01snd83q.jpg"><img alt="image" src="https://64.media.tumblr.com/7358668d0602c3cba7e99a4bcbc650f6/tumblr_inline_pk317lzF8w1snd83q_540.jpg" data-orig-height="605" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/5f171945926c2ff6369d4b2f73b75e87/tumblr_inline_nbwm74ZbD01snd83q.jpg"></figure>

I spent £12 on a small kitchen scale which can weigh objects up to 3kg. I taped 2kg worth of books to the quadcopter but I could only test the 200-900ms range before it would fall from the scale. ASDA makes a [fantastic £8 kitchen scale](https://direct.asda.com/George-Home-Electronic-Flat-Scales-10kg/001468048,default,pd.html) which can weigh up to 10kg. I taped the quadcopter to a 8kg box of books and I could measure the whole 200-2000ms range without any problem.

<figure class="tmblr-full" data-orig-height="334" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/da466d24e1ebb7c06c8e5667c04d8def/tumblr_inline_nbwpynhPMi1snd83q.jpg"><img alt="image" src="https://64.media.tumblr.com/06e81d4064517c013abf03a25fb1b5c5/tumblr_inline_pk317lyAM91snd83q_540.jpg" data-orig-height="334" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/da466d24e1ebb7c06c8e5667c04d8def/tumblr_inline_nbwpynhPMi1snd83q.jpg"></figure>

In order to acquire as many samples as fast as possible, I setup a camera on a tripod to take photos of the scale and the computer screen as I increased the motor speed. I ran two data acquisition rounds: each produced more than 100 samples in 3-4 minutes. I noted the unloaded battery voltage at the end of the measure to differentiate the data sets: these are not the voltages under load while the motor was spinning.

<figure class="tmblr-full" data-orig-height="295" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/fecd3757f4cccab05d8644382c150665/tumblr_inline_nbwrzpL0wa1snd83q.jpg"><img alt="image" src="https://64.media.tumblr.com/0eaa5b02570eac66c60c4b08cc87d5fa/tumblr_inline_pk317mJf8o1snd83q_540.jpg" data-orig-height="295" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/fecd3757f4cccab05d8644382c150665/tumblr_inline_nbwrzpL0wa1snd83q.jpg"></figure>

**Analysis**

[Apparently](https://andrew.gibiansky.com/blog/physics/quadcopter-dynamics/) the laws of physics predict that the force magnitude should be proportional to the square of the RPMs. The SimonK ESC firmware claims to have a linear throttle response, which I suppose is meant to be understood in terms of RPMs. If this claim (and physics) are true, then the square root of the lift force could be expressed as a linear function of the PWM. This turns out to be a reasonable estimate, especially if we exclude the extremely high and low PWM values.

<figure class="tmblr-full" data-orig-height="333" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/5d4ff62cc22141623a58dbb04fe29918/tumblr_inline_nbwvt4ihtJ1snd83q.jpg"><img alt="image" src="https://64.media.tumblr.com/a84ac2604562042a814f54562ae9c2e7/tumblr_inline_pk317mUgmd1snd83q_540.jpg" data-orig-height="333" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/5d4ff62cc22141623a58dbb04fe29918/tumblr_inline_nbwvt4ihtJ1snd83q.jpg"></figure>

f(x) = 0.00118415\*x + 0.30756716

The linear RPMs estimate leads to the following quadratic force estimate. Again it's not too bad, especially if we exclude the extremely high PWM values.

<figure class="tmblr-full" data-orig-height="277" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/6d1390c03faded45768ee2889f9e6e48/tumblr_inline_nbwwoaU7wp1snd83q.jpg"><img alt="image" src="https://64.media.tumblr.com/1239be49704b7e8934ddb60cf8a1d838/tumblr_inline_pk317mp0NO1snd83q_540.jpg" data-orig-height="277" data-orig-width="500" data-orig-src="https://64.media.tumblr.com/6d1390c03faded45768ee2889f9e6e48/tumblr_inline_nbwwoaU7wp1snd83q.jpg"></figure>

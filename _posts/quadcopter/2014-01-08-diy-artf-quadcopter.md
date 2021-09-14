---
layout: post
title: DIY ARTF quadcopter
date: '2014-01-08T00:04:00+08:00'
tags:
- quadcopter
- frame
- esc
- simonk
- motor
---
{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2014-01-08-diy-artf-quadcopter-1.jpg)
{:refdef}

eBay basket:

- £4.99 18mm x 10mm x 300mm craft sticks (12x)
- £2.00 100mm x 100mm x 5mm perspex sheet
- £3.95 M5 bolts + washers + nuts (10x)
- £5.69 16x19 brushless motor mount (4x)
- £3.95 power distribution board
- £41.19 F-30A ESC flashed with SimonK firmware (4x)
- £43.50 AGM MT2213 935kV motor (4x)

The cost is £105.27 in total, or £24.53 for the frame only. In comparison an ARTF DJI F450 costs £130-140, or £30-35 for the frame only.

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2014-01-08-diy-artf-quadcopter-2.jpg)
{:refdef}

I separately tested each ESC connected to the throttle channel of a receiver. I successfully went through the following initialisation sequences described in [the doc](https://github.com/sim-/tgy/blob/master/README.md).

**Normal startup**

- Connect battery with throttle stick at min
- ESC emits a long beep, we're ready to go

**Calibration startup**

- Connect battery with throttle stick at max
- ESC emits a short beep to indicate it saved the max throttle position
- Move throttle stick to min
- ESC emits two short beeps to indicate it saved the min throttle position
- ESC emits a long beep, we're ready to go

After the startup sequence the throttle stick controls the speed of the attached motor. Back to working on the software now!

---
layout: post
title: PS2 controller pinout
date: '2014-10-23T23:41:00+08:00'
tags:
- ps2
- controller
- wire
- color
---
I would like to use a PS2 controller with the quadcopter and maybe future projects. The analog sticks may not be as precise as a dedicated RC transmitter, but a dual shock is less thank half the size and much more durable for traveling.

I purchased a controller extension lead. I cut the cable in half: I'm only interested in the controller side. I then crushed the console side with a vise grip to identify the wires colors. The colors might differ from cable to cable, don't trust my findings.

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2014-10-23-ps2-controller-pinout-1.jpg)
{:refdef}

From left to right, based on [Curious Inventor](https://store.curiousinventor.com/guides/PS2/)'s pinout:

- Green: Acknowledge
- N/C
- Blue: Clock
- Yellow: Attention
- Red: Power (3.3V)
- Black: Ground
- Grey: Motors power
- Orange: Command
- Brown: Data

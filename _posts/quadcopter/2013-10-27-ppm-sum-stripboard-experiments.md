---
layout: post
title: PPM sum stripboard experiments
date: '2013-10-27T02:18:00+08:00'
tags:
- stripboard
- ppmsum
- rc
---
I got a 6 channels OrangeRX radio receiver for £7.99 on eBay.

The way RC receivers work is that each channel emits a pulse which lasts between 1ms and 2ms every 20ms (50Hz). This is exactly what servos expect, which simplifies much of the RC vehicles electronics.

In order to decode the 6 channels we could use 6 Arduino pins. There is however a trick with the OrangeRX receiver and many others. Each channel emits its pulse at a different time. The 20ms cycle is divided in eight 2.5ms slots, and each slot is assigned to a channel. We could merge all the channels through an OR gate and use only one Arduino pin to decode them all. The merged signal is called the PPM sum.

I could have purchased a ready-made PPM encoder for less than £10, but I thought it was an opportunity to experiment with hardware. I ordered an [8-inputs OR gate](https://www.spiratronics.com/data/6645.pdf) and a stripboard. I had never used a stripboard, yay!

I first experimented on the breadboard. I discovered that **all** the inputs of the OR gate have to be connected. If not the output oscillates. I connected the two extra inputs to the ground and everything worked as expected.

I then realised a first stripboard design.

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2013-10-27-ppm-sum-stripboard-experiments-1.jpg)
{:refdef}

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2013-10-27-ppm-sum-stripboard-experiments-2.jpg)
{:refdef}

It worked well but I realised a few improvements could be made.

- I should have thought about the pin layout better. Two pins for the power and a single pin for the PPM output do not provide enough friction to secure the wires and the board together.
- I should have used right-angled headers instead of straight ones, they help get a cleaner wiring around the board.
- The pins 10 and 11 of the 4078 are connected to the ground. They could have been connected to the pins 4 and 5 instead. This would have saved two holes and two wires.

With this in mind I created a new version of the stripboard.

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2013-10-27-ppm-sum-stripboard-experiments-3.jpg)
{:refdef}

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2013-10-27-ppm-sum-stripboard-experiments-4.jpg)
{:refdef}

I swapped the two bottom tracks and moved the PPM output to provide servo wire compatibility on the Arduino side. I also added a pin on the receiver side to provide more friction and allow the use of 3 adjacent servo wires. The wiring is now much better looking and there is enough friction to secure the wires in place.

I'm glad I didn't purchase an off the shelf PPM sum board, this was a great introduction to DIY electronics.

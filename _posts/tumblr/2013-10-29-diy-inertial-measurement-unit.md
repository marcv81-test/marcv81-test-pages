---
layout: post
title: 'DIY IMU - Part 1: build'
date: '2013-10-29T00:50:00+08:00'
tags:
- imu
- accelerometer
- magnetometer
- python
- arduino
- stripboard
- gyroscope
tumblr_url: https://robokitchen.tumblr.com/post/65387650416/diy-inertial-measurement-unit
---
Now that we can read from the accelerometer/gyroscope and the magnetometer we have all we need to build an Inertial Measurement Unit (IMU).

The first step is to secure the MPU-6050 and the HMC5883L together. For this I built a tiny 8x4 stripboard. With the breakout boards GY-521 and GY-273 it’s easy to align VCC, GND, SCL, and SDA. The pin headers need to be soldered in a way which allows this. I soldered mines so that the pins are on the side with no components on both breakout boards. I also punched 3mm holes in a blank plastic card and attached the sensors with M3 nylon screws. This way there is no relative movement between the MPU-6050 and the HMC5883L, and their axes are reasonably well aligned.

<figure data-orig-height="222" data-orig-width="280"><img alt="image" src="https://64.media.tumblr.com/4068e4e9ce2def8328cd8d6e32bb265f/d452b2cfbe3bfdf6-ee/s540x810/d3147d149b34c1fee16073c49f83fe12b2085671.jpg" data-orig-height="222" data-orig-width="280"></figure>

<figure class="tmblr-full" data-orig-height="300" data-orig-width="400"><img alt="image" src="https://64.media.tumblr.com/0e4bf73cff64ab5492205508bfdd2104/d452b2cfbe3bfdf6-a6/s540x810/30aa8ad53793500b8589e67ab3190bdd050ca8ce.jpg" data-orig-height="300" data-orig-width="400"></figure>

We shall now agree on an [axes convention](http://en.wikipedia.org/wiki/Axes_conventions). Most aircrafts use the following, so we’ll use that as well.

- The X axis points ahead
- The Y axis points to the right
- The Z axis points downwards

I decided that the front of the IMU would be opposite to where the cables go, and that the photo above shows the IMU the right side up. According to the drawings on the breakout boards, the X and Y accelerometer axes already follow our convention. For the magnetometer we need to reverse the direction of the Y axis then swap the X and Y axes. I implemented this with [preprocessor macros](https://github.com/marcv81/robokitchen/blob/322f3bd33da00e50fe8f5c1d2fc6bed426e20a90/libraries/IMU/IMU.h). Neither of the breakout boards have the Z axis drawn so we’ll have to determine its direction experimentally.

I wrote a&nbsp;[new sketch](https://github.com/marcv81/robokitchen/blob/322f3bd33da00e50fe8f5c1d2fc6bed426e20a90/sketches/RawIMUTest/RawIMUTest.ino) to output the accelerometer and magnetometer data. It works well in the Arduino serial monitor, but my brain struggles to process six floating point numbers every few milliseconds. I hence decided to plot a somehow more graphical representation of the data. I saw a number of good demos in [Python](http://www.python.org/) so I was eager to give it a try. I installed the main package and two extra components: [VPython](http://www.vpython.org/) and [pySerial](http://pyserial.sourceforge.net/). I’m new to Python but it’s all very simple: I wrote a script to display the accelerometer vector in blue and the magnetometer vector in green and it took less than [20 lines](https://github.com/marcv81/robokitchen/blob/322f3bd33da00e50fe8f5c1d2fc6bed426e20a90/python/RawIMUTest/RawIMUTest.py). The only caveat is that pySerial or VPython do not seem to manage a constant stream of data at 115200 baud too well, so we included a 100ms delay in the sketch.

<figure class="tmblr-full" data-orig-height="451" data-orig-width="430"><img alt="image" src="https://64.media.tumblr.com/92d32bb4e8520e98fed15e0019f1269c/d452b2cfbe3bfdf6-a1/s540x810/5fe3984e40999610f77eab9724144547017b9406.jpg" data-orig-height="451" data-orig-width="430"></figure>

The IMU is not subject to any other significant acceleration than the gravity so the [accelerometer](http://en.wikipedia.org/wiki/Accelerometer) mostly measures the gravity vector. For some reason the gravity is upward. The IMU is not in proximity to important sources of magnetic interferences, so the [magnetometer](http://en.wikipedia.org/wiki/Magnetometer) mostly measures the Earth’s magnetic field vector. I’m in London where the [magnetic dip](http://www.geomag.bgs.ac.uk/data_service/models_compass/wmm_calc.html) is about 66 degrees.

The first try did not give the expected results: the angle between the vectors did not remain constant as expected. This was a problem with the axes. I reviewed the [HMC5883L registers map](http://www51.honeywell.com/aero/common/documents/myaerospacecatalog-documents/Defense_Brochures-documents/HMC5883L_3-Axis_Digital_Compass_IC.pdf) and realised that I had swapped the Y and Z axes. Once this was [fixed](https://github.com/marcv81/robokitchen/commit/ad3171cef16887ff457030b971b38fec3f5d2374) everything else worked fine.

We now have a reliable way to measure the accelerometer and magnetometer vectors. If the gyroscope works too we should be able to combine the data to determine the IMU attitude.


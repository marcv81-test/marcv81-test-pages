---
layout: post
title: 'Integration - Part 1: IMU'
date: '2014-04-24T02:26:09+08:00'
tags:
- quadcopter
- imu
- motors
- vibrations
- bluetooth
- axes
- dlpf
---
The IMU algorithm has been working great with separate sensors and under ideal circumstances. Let's try to get it to work using the MultiWii board and with the motors running.

**Axes**

First I checked the IMU, motors off, using the USB serial port. I tried to visualise the orientation but the axes were incorrect. I naively expected the Multiwii board to have its axes aligned with the little painted arrow but it's not the case.

I first tried to look at the Multiwii source code but it's weird. I know the MPU-6050 accelerometer and gyroscope axes are aligned in the hardware, however the [Multiwii code](https://code.google.com/p/multiwii/source/browse/trunk/MultiWii_shared/def.h) defines different axes mapping for each (for CRIUS_SE_v2_0 for instance). This means Multiwii uses different axes for the accelerometer and the gyroscope: I'd be curious if someone had a good explanation!

After a few tries I figured the following.

- X => Y
- Y => X
- Z => -Z

**Motors on**

We cannot use the USB serial port when the motors are running (we would be connecting power supplies with possible voltage differences in parallel). We'll use the Bluetooth serial port instead. I [modified the Python scripts](https://github.com/marcv81/quadcopter/commit/d3f3c4523c3501669b161e4894646e9e84902e95) so that the serial port and the baud rate are specified as command line arguments. This allows to switch between USB and Bluetooth easily.

To visualise the orientation over Bluetooth:

- Run _stty -f /dev/tty.Robokitchen-DevB 115200_ (in OSX)
- Run _Orientation.py /dev/tty.Robokitchen-DevB 155200_ (in OSX)

**Vibrations**

The first obvious observation is that the orientation is off when the motors are running. This is probably because of vibrations.

I [enabled the MPU6050 digital low pass filter (DLPF)](https://github.com/marcv81/quadcopter/commit/38a3554b2875375a8c083a5be7cb6f30efa31421) at its most aggressive setting: 5Hz cut-off frequency. At first I thought this would be pointless because the IMU complementary filter already filtered out the accelerometer high-frequencies (i.e.: vibrations), but the DLPF removes vibrations form both the gyroscope and the accelerometer. This brings back the orientation estimation to a useable accuracy.

The next step will be to implement controllers to drive the motors based on the orientation estimation and its desired value.

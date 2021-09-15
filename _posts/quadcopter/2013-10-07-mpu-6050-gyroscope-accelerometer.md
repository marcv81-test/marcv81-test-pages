---
layout: post
title: MPU-6050 gyroscope/accelerometer
date: '2013-10-07T22:13:00+08:00'
tags:
- arduino
- gyroscope
- accelerometer
- mpu6050
---
The first step to communicate with the MPU-6050 is to setup the I2C bus. The official TWI/Wire library has poor reviews so I tried the one from [DSS Circuits](https://www.dsscircuits.com/articles/arduino-i2c-master-library.html) instead. I unpacked the library into [libraries/contrib/I2C](https://github.com/marcv81/quadcopter/tree/061e2fd229f7f7cb083cbfa6e5aabdc661e4131d/libraries/contrib/I2C) for versioning purposes and copied it to ~/Documents/Arduino/libraries to be able to include it in my sketches.

As a quick test I created a [new sketch](https://github.com/marcv81/quadcopter/blob/9ed9d4e792d77aa62858570bf557dabb549d7868/sketches/I2CTest/I2CTest.ino) to read the WHO_AM_I register. It worked without any problem, easy!

I then wanted to read actual accelerometer/gyroscope data. The [MPU-6050 Register Map](https://www.invensense.com/mems/gyro/documents/RM-MPU-6000A.pdf) tells us all we need to know. I went through most of the descriptions and chose the following initialisation sequence.

1. Reset the device to the default settings
2. Disable the sleep mode

It's basic but we don't need anything more for now. The accelerometer is on the +/- 2G setting, with a sensitivity of 16384 LSB/G. The gyroscope is on the +/- 250 °/s setting, with a sensitivity of 131 LSB/°/s.

Reading 14 bytes from the register 0x3B gives us in this order: 3x 16 bits of accelerometer data, 1x 16 bits of temperature data, and 3x 16 bits of gyroscope data. The MPU-6050 is big-endian and the Arduino is little-endian, so we need to swap the MSB and LSB of each 16 bits value.

I put all this into a [library](https://github.com/marcv81/quadcopter/tree/3d1f3dbaea2709146d5706d666cc74b6ff8518d6/libraries/MPU6050) and its companion [test sketch](https://github.com/marcv81/quadcopter/blob/3d1f3dbaea2709146d5706d666cc74b6ff8518d6/sketches/MPU6050Test/MPU6050Test.ino). Below are the first readings: I started at rest, then moved the sensor board, and ended at rest again.

**Accelerometer**

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2013-10-07-mpu-6050-gyroscope-accelerometer-1.jpg)
{:refdef}

At rest the Z axis reads -1G. The expected value is 1G which simply means the [accelerometer](https://en.wikipedia.org/wiki/Accelerometer) is upside down. The X and Y axis are not exactly at 0 because the sensor is not perfectly horizontal.

**Gyroscope**

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2013-10-07-mpu-6050-gyroscope-accelerometer-2.jpg)
{:refdef}

At rest all the axis are at 0 as expected.

All the values seem to be correct. We are now in a position to compute the orientation from the accelerometer and gyroscope (and possibly magnetometer) data.

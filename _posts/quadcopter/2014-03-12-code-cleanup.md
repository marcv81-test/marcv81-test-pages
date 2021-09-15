---
layout: post
title: Code cleanup
date: '2014-03-12T23:03:00+08:00'
tags:
- arduino
- imu
- cleanup
---
I uncluttered the repository a bit. I removed the sketches which were no longer useful, and more importantly I [merged all the IMU-related sketches and scripts](https://github.com/marcv81/quadcopter/commit/d5a63040bf637d052b2a674e64d2160c906286ae) together. To visualise any of the IMU data just upload IMUTest.

- python/Orientation.py: displays the IMU forward and down vectors in the fixed reference frame
- python/Acceleration.py: displays the acceleration vector in the IMU reference frame
- python/Magnetometer.py: displays the magnetometer vector in the IMU reference frame

The IMU is configured using the following flags:

- IMU_ACCEL_ENABLE: if defined the accelerometer is used as a fixed reference vector to prevent attitude drift
- IMU_MAGNET_ENABLE: if defined the magnetometer is used as a fixed reference vector to prevent attitude drift
- IMU_GYRO_CALIBRATION: if defined the gyroscope is dynamically calibrated when the sketch starts
- IMU_MAGNET_CALIBRATION: if defined the magnetometer is statically calibrated using the output of octave/magnet_calibration/generate_header.m

The steps to calibrate the magnetometer are:

- Upload sketch-test/IMUTest with the magnetometer calibration disabled
- Run: _python Magnetometer.py > ../../octave/magnet_calibration/data.csv_
- In octave/magnet_calibration run: _octave -q generate_header.m_
- Upload sketch-test/IMUTest with the output of the previous command in the config.h

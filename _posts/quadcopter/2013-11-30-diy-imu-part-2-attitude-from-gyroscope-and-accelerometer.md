---
layout: post
title: 'DIY IMU - Part 2: attitude from gyroscope and accelerometer'
date: '2013-11-30T21:16:00+08:00'
tags:
- imu
- quaternion
- attitude
- arduino
- accelerometer
- gyroscope
---
We are getting data from the accelerometer, the magnetometer, and the gyroscope. Actually the magnetometer suffers from [soft and hard iron distortions](https://www.vectornav.com/support/library?id=83) so we'll leave it for later. With just the gyroscope and the accelerometer we already have enough data to calculate the attitude of the IMU. Let's define what this means.

We have two [reference frames](https://en.wikipedia.org/wiki/Axes_conventions): a local reference frame attached to the IMU, and a fixed reference frame (attached to the Earth). Let's use the {x=North, y=East, z=down} coordinate system for the fixed reference frame. Let's use the {x=forward, y=right, z=down} coordinate system for the local reference frame. When the IMU is upright and points North the fixed and local reference frames are aligned.  
  
The attitude is the rotation of the local reference frame observed in the fixed reference frame.

[Quaternions](https://en.wikipedia.org/wiki/Quaternions_and_spatial_rotation) can represent rotations in 3D space. We'll calculate the attitude as a quaternion because it is computationally inexpensive in comparison to other representations.

**Attitude from gyroscope**

The gyroscope measures the angular velocity on each axis. We can calculate the rotation angle between two measures on each axis multiplying the angular velocity by the time elapsed. This gives us the [axis-angle](https://en.wikipedia.org/wiki/Axis%E2%80%93angle_representation) representation of the rotation between two measures.

We'll assume that when we power the IMU the fixed reference frame and the local reference frame are aligned. The attitude is the total rotation measured by the gyroscope. We can calculate it accumulating all the rotations between two measures. There is no easy formula to combine rotations using the axis-angle representation. The first step is to convert them to quaternions using [this formula](https://en.wikipedia.org/wiki/Axis%E2%80%93angle_representation#Unit_Quaternions). The Hamilton product of two quaternions represents the combination of two rotations. If $$q(t-1, t)$$ is the rotation between t-1 and t, then the total rotation $$q_{total}$$ can be expressed recursively as $$q_{total}(t) = q(t-1, t) * q_{total}(t-1)$$.

There are two caveats to consider.

- The gyroscope has a bias: even when not moving the angular velocity on each axis is not null. We can calibrate the gyroscope calculating the bias upon powering the IMU. We then subtract the bias to the measured values during normal operation. This is not perfect because the bias might change over time, so small errors remain (sometimes large errors, for instance after a shock).
- Small errors accumulated over time eventually result in large errors: we can compensate the error using an accelerometers and/or a magnetometer.

**Error correction from accelerometer**

If we rotate the coordinates of a vector in the fixed reference frame by the **opposite** of the attitude we obtain the coordinates of the same vector in the local reference frame. This is worth a quick explanation. Let's imagine a compass laying flat on a table with the fixed and local reference frames aligned. The needle reads 0 degrees. Let's now turn the compass 45 degrees clockwise. The attitude is 45 degrees clockwise. However from the perspective of the compass base plate the needle turned 45 degrees counterclockwise: the needle reads 45 degrees counterclockwise. Again, to get the coordinates of a vector in the local reference frame we apply the **opposite** of the attitude to its coordinates in the fixed reference frame.

The accelerometer measures in the local reference frame the gravity vector which is assumed known an invariant in the fixed reference frame. If we rotate the coordinates of this vector in the fixed reference frame by the opposite of the gyroscope attitude we get its expected coordinates in the local reference frame according to the gyroscope attitude.

The rotation from the expected vector to the measured vector is the error between the gyroscope attitude estimate and the accelerometer measurement. We can calculate this error in the axis-angle representation using the [formula from a previous post](https://robokitchen.tumblr.com/post/68484965839/finding-the-angle-axis-rotation-between-two-vectors). When calculated this way, the error coordinates are in the local reference frame.

This error is due to a combination of the following factors.

- Accelerometer measurement errors (high-frequency): the accelerometer may be imprecise and is sensitive to other sources of acceleration than the gravity.
- Gyroscope drift (low-frequency): small errors accumulated over time eventually result in large errors.

We want to compensate for the gyroscope drift but obviously we don't want to pick up noise from the accelerometer measurement errors. This can be achieved with a digital low-pass filter.

Let's subtract a fraction of the accelerometer error from the gyroscope measure. We can do this because both quantities are in the axis-angle representation and in the local reference frame. If the error is high-frequency (i.e.: happens once) we will only pick a fraction of it once with virtually no consequence. If the error is low-frequency (i.e.: happens regularly for a number of cycles) we will pick enough fractions to compensate for it eventually. This is exactly what we want.

**Implementation**

All the above got implemented in a [new sketch, libraries, and a python script](https://github.com/marcv81/quadcopter/commit/054294b91dfcd15f7b4e27477f1191cab42ce879) to visualise the results as forward and down vectors. Everything works nicely and fast: 1.5ms per cycle without the accelerometer, 2.5ms with it. I had to increase the [gyroscope scale](https://github.com/marcv81/quadcopter/commit/c2154e0fea6917a0d3e4e751fef5fc0b8bea784f) to be able to keep track of the attitude when the IMU moves fast. There is a small drift on the z axis, but we can't really fix that without the magnetometer.

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2013-11-30-diy-imu-part-2-attitude-from-gyroscope-and-accelerometer-1.jpg)
{:refdef}

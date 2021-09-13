---
layout: post
title: 'DIY IMU - Part 3: magnetometer integration'
date: '2014-01-17T00:18:00+08:00'
tags:
- imu
- magnetometer
- arduino
tumblr_url: https://robokitchen.tumblr.com/post/73560236354/i-had-in-mind-to-calibrate-the-magnetometer-for-a
---
So far the IMU works well to estimate the pitch and the roll but we still have a slight yaw drift. This can be fixed including the magnetometer data in our equations, but we have to calibrate the device first.

The magnetometer can be calibrated for [hard and soft iron distortions](http://memsblog.wordpress.com/2011/03/22/hard-and-soft-iron-magnetic-compensation-explained/). If there was no distortion and we turned the magnetometer toward different directions, the values read should describe a centered sphere. The hard iron distortion moves the sphere off the center, and the soft iron distortion changes its shape into that of an ellipsoid. The aim of the calibration is to find a transformation to compensate for these distortions and map the magnetometer values back to a centered sphere.

There wasn’t very much distortion around my sensor, so I mounted a few ferrous and magnetic items next to it: a steel maillon and a few sevos. This shall help testing that the following steps work indeed.

Before anything we have to improve the HMC5883L library. The magnetometer updates at 15Hz, and we are likely to read its data faster than that. The refresh() function may return the same values more than once, which we’d rather avoid. I tried the single-measurement mode with very little success and finally left the device in continuous-measurement mode. I [changed the library](https://github.com/marcv81/robokitchen/commit/a24c8e393e1ef1e1b1de78a4c21eea2f2f0f929a) to make sure the values are only returned once. In fact we read the values anyway and compare them with the previous ones: if they are the same refresh() returns 0, if they are different we update the magnetometer values and return 1.

Now we need to capture the magnetometer data. I created a [new sketch](https://github.com/marcv81/robokitchen/commit/d76a88d8c96914c3fc5b4e54331cef7d5241450b) and a [python script](https://github.com/marcv81/robokitchen/commit/0d308edbf5c7d96256960f7ee60a3955ab571649) to visualise the acquired data. The script displays an increasing large cloud of points, which lets us know the progress. Below are the plotted results for my setup.

<figure class="tmblr-full" data-orig-height="446" data-orig-width="425" data-orig-src="https://64.media.tumblr.com/bf4c293129a5d03da5b13eb42a84115c/tumblr_inline_mziltc2kTv1snd83q.jpg"><img alt="image" src="https://64.media.tumblr.com/bf4c293129a5d03da5b13eb42a84115c/tumblr_inline_pl1jjqWNv11snd83q_540.jpg" data-orig-height="446" data-orig-width="425" data-orig-src="https://64.media.tumblr.com/bf4c293129a5d03da5b13eb42a84115c/tumblr_inline_mziltc2kTv1snd83q.jpg"></figure>

_python MagnetometerCalibration.py \> data.csv_

Yury Petrov wrote an [Octave script](http://www.mathworks.co.uk/matlabcentral/fileexchange/24693-ellipsoid-fit) which computes the parameters of the closest ellipsoid to a list of points using the least mean squares method. I don’t have Octave, let’s install it via Macports. The official documentation recommends to run “_sudo port install octave +atlas+docs_” but that didn’t work for me. I ran “_sudo port install octave +atlas_” which did the trick.

The script returns the following ellipsoid parameters.

- Ellipsoid center $\vec{v}$  
- Ellipsoid radii&nbsp;$\vec{r} = \begin{bmatrix}r\_a \\ r\_b \\ r\_c \end{bmatrix}$
- Ellipsoid radii vectors $\vec{a}$, $\vec{b}$, $\vec{c}$ in matrix form $R = \begin{bmatrix}a\_x & b\_x & c\_x \\ a\_y & b\_y & c\_y \\ a\_z & b\_z & c\_z\end{bmatrix}$

**Hard iron distortion**

In order to compensate for the hard iron distortion we simply subtract $\vec{v}$ from the magnetometer vector $\vec{x}$. The corrected value is $\vec{x}^\prime = \vec{x} - \vec{v}$.

**Soft iron distortion**

In the reference frame defined by $\vec{a}$, $\vec{b}$, $\vec{c}$, the centered ellipsoid (i.e.: after correcting the hard iron distortion) is aligned with the axes. In this reference frame the scaling transformation to map the aligned ellipsoid to a sphere can be written as follows.

\[S\_{abc} = \begin{bmatrix}1/r\_a & 0 & 0 \\ 0 & 1/r\_b & 0 \\ 0 & 0 & 1/r\_c\end{bmatrix}\]

By definition R is the rotation matrix which transform the axes $\vec{x}$, $\vec{y}$, $\vec{z}$ into $\vec{a}$, $\vec{b}$, $\vec{c}$. The transpose matrix $R^T$ is the opposite rotation. So the scaling matrix in the $\vec{x}$, $\vec{y}$, $\vec{z}$ reference frame can be written as follows.

\[S = R \times ( S\_{abc} \times R^T)\]

The corrected value is $\vec{x}^{\prime\prime} = S \times \vec{x}^\prime = S \times (\vec{x} - \vec{v})$.

I included the above math in an [Octave script](https://github.com/marcv81/robokitchen/commit/aa85b2a5960ee59be0ebe3623306059822be1190) which generates a configuration header containing 12 parameters: 3 for the hard iron vector and 9 for the soft iron matrix.

_octave -q generate\_header.m \> config.h_

Then I added a [new library](https://github.com/marcv81/robokitchen/commit/459d7193e48b6da028f21ac136d73470d3bb7970) to apply the transformations and map the magnetometer vector to a sphere. Below are the plotted results of the magnetometer data capture sketch modified to use the sphere mapping.

&nbsp;<figure class="tmblr-full" data-orig-height="446" data-orig-width="425" data-orig-src="https://64.media.tumblr.com/381ddc54ee5c085b2010f1539b4530b5/tumblr_inline_mzind16su41snd83q.jpg"><img alt="image" src="https://64.media.tumblr.com/381ddc54ee5c085b2010f1539b4530b5/tumblr_inline_pl1jjr3vzW1snd83q_540.jpg" data-orig-height="446" data-orig-width="425" data-orig-src="https://64.media.tumblr.com/381ddc54ee5c085b2010f1539b4530b5/tumblr_inline_mzind16su41snd83q.jpg"></figure>

The magnetometer data is finally useable as a fixed heading reference. We can use it to correct the attitude yaw drift.

We are going to use almost exactly the same strategy as with the accelerometer.

- First we rotate the North vector (1, 0, 0) in the fixed reference frame by the conjugate of the attitude. This is the gyroscope estimate of the North vector in the local reference frame.
- We apply the same rotation to the up vector (0, 0, -1). We then calculate the projection of the magnetometer corrected vector onto the plane orthogonal to the up vector. This is the magnetometer estimate of the North vector in the local reference frame.
- The difference between the two estimates is the error, which we can correct with the same formulas we used for the accelerometer.

We project the magnetometer vector onto the horizontal plane to make sure we only corrects the yaw. The accelerometer is already very precise to correct the roll and pitch, and we would not want inconsistencies in the local magnetic field to crash a quadcopter.

After [implementing the above](https://github.com/marcv81/robokitchen/commit/f27cf5b793320f0f34ace6192b10959c8b1d703f) our IMU can estimate the attitude without any drift. Each cycle runs in less than 3ms.


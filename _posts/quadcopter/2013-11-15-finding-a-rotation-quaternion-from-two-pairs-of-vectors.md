---
layout: post
title: Finding a rotation quaternion from two pairs of vectors
date: '2013-11-15T15:14:00+08:00'
tags:
- math
- vector
- quaternion
- rotation
---
This post discusses a fast quaternion-based algorithm to determine the rotation between two pairs of non-collinear vectors.

**(EDIT)** This algorithm can be used to estimate the attitude quaternion from an accelerometer and a magnetometer. It is possible to interpolate this quaternion with the attitude quaternion found integrating the gyroscope rate of change. This does eliminate the gyroscope drift but it's a suboptimal way to compute the attitude quaternion from all the sensors. The speed and accuracy are good, but more flexible algorithms allow to correct the gyroscope attitude using only an accelerometer, only a magnetometer, or both. I no longer use the algorithm described here.

If you're still reading here is the algorithm description and explanation.

- $$u_0$$ and $$v_0$$ are the two vectors before the rotation
- $$u_2$$ and $$v_2$$ are the two vectors after the rotation

We are looking for the rotation q which verifies the following.

- \$$u_0 \xrightarrow{q} u_2$$
- \$$v_0 \xrightarrow{q} v_2$$

We assume that such a rotation exists. This is not generally the case.

Let's decompose q into two successive rotations $$q_1$$ and $$q_2$$.

- \$$u_0 \xrightarrow{q_1} u_1 \xrightarrow{q_2} u_2$$
- \$$v_0 \xrightarrow{q_1} v_1 \xrightarrow{q_2} v_2$$

Let's choose $$q_1$$ such that $$u_0 = u_1$$. We deduce the following.

- $$q_1$$ is a rotation around the $$u_0$$ axis.
- $$q_2$$ is a rotation which transforms $$u_0$$ into $$u_2$$.

We calculate $$q_2$$ from $$u_0$$ and $$u_2$$ using [this formula](https://lolengine.net/blog/2013/09/18/beautiful-maths-quaternion-from-vectors).  
  
We then apply the opposite of $$q_2$$ rotation (i.e.: the conjugate of the quaternion) to $$v_2$$ to obtain $$v_1$$ using [this formula](https://molecularmusings.wordpress.com/2013/05/24/a-faster-quaternion-vector-multiplication/).  
  
We know the following about $$q_1$$.

- $$q_1$$ is around the $$u_0$$ axis.
- $$q_1$$ transforms $$v_0$$ into $$v_1$$.

We cannot calculate $$q_1$$ from $$v_0$$ and $$v_1$$ using the formula we used to find $$q_2$$. It may not return a rotation around the $$u_0$$ axis.

Let's call $$v_0^\prime$$ and $$v_1^\prime$$ the projections of $$v_0$$ and $$v_1$$ on the plane orthogonal to $$u_0$$. We calculate $$v_0^\prime$$ and $$v_1^\prime$$ using [this formula](https://en.wikipedia.org/wiki/Vector_projection).

The rotation $$q_1$$ around $$u_0$$ which transforms $$v_0$$ into $$v_1$$ also transforms $$v_0^\prime$$ into $$v_1^\prime$$. I can't prove it formally but it's visually obvious.

We calculate $$q_1$$ from $$v_0^\prime$$ and $$v_1^\prime$$ using the formula we used to find $$q_2$$. The axis of $$q_1$$ will necessarily be $$u_0$$ because $$v_0^\prime$$ and $$v_1^\prime$$ are in a plane orthogonal to $$u_0$$.

We finally calculate q = $$q_2 q_1$$.

If the vectors come from error-prone measurements q may not exist. In this case the algorithm provides a quaternion which transforms $$u_0$$ into $$u_2$$ and $$v_0$$ into a vector "close enough" to $$v_2$$. The algorithm is hence biased towards one of the pairs of vectors.

The bias is a tradeoff for a fast and fixed execution time. On some applications it might actually be an advantage.

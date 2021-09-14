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

- $u\_0$ and $v\_0$ are the two vectors before the rotation
- $u\_2$ and $v\_2$ are the two vectors after the rotation

We are looking for the rotation q which verifies the following.

- $u\_0 \xrightarrow{q} u\_2$
- $v\_0 \xrightarrow{q} v\_2$

We assume that such a rotation exists. This is not generally the case.

Let's decompose q into two successive rotations $q\_1$ and $q\_2$.

- $u\_0 \xrightarrow{q\_1} u\_1 \xrightarrow{q\_2} u\_2$
- $v\_0 \xrightarrow{q\_1} v\_1 \xrightarrow{q\_2} v\_2$

Let's choose $q\_1$ such that $u\_0 = u\_1$. We deduce the following.

- $q\_1$ is a rotation around the $u\_0$ axis.
- $q\_2$ is a rotation which transforms $u\_0$ into $u\_2$.

We calculate $q\_2$ from $u\_0$ and $u\_2$ using [this formula](https://lolengine.net/blog/2013/09/18/beautiful-maths-quaternion-from-vectors).  
  
We then apply the opposite of $q\_2$ rotation (i.e.: the conjugate of the quaternion) to $v\_2$ to obtain $v\_1$ using [this formula](https://molecularmusings.wordpress.com/2013/05/24/a-faster-quaternion-vector-multiplication/).  
  
We know the following about $q\_1$.

- $q\_1$ is around the $u\_0$ axis.
- $q\_1$ transforms $v\_0$ into $v\_1$.

We cannot calculate $q\_1$ from $v\_0$ and $v\_1$ using the formula we used to find $q\_2$. It may not return a rotation around the $u\_0$ axis.

Let's call $v\_0^\prime$ and $v\_1^\prime$ the projections of $v\_0$ and $v\_1$ on the plane orthogonal to $u\_0$. We calculate $v\_0^\prime$ and $v\_1^\prime$ using [this formula](https://en.wikipedia.org/wiki/Vector_projection).

The rotation $q\_1$ around $u\_0$ which transforms $v\_0$ into $v\_1$ also transforms $v\_0^\prime$ into $v\_1^\prime$. I can't prove it formally but it's visually obvious.

We calculate $q\_1$ from $v\_0^\prime$ and $v\_1^\prime$ using the formula we used to find $q\_2$. The axis of $q\_1$ will necessarily be $u\_0$ because $v\_0^\prime$ and $v\_1^\prime$ are in a plane orthogonal to $u\_0$.

We finally calculate q = $q\_2 q\_1$.

If the vectors come from error-prone measurements q may not exist. In this case the algorithm provides a quaternion which transforms $u\_0$ into $u\_2$ and $v\_0$ into a vector "close enough" to $v\_2$. The algorithm is hence biased towards one of the pairs of vectors.

The bias is a tradeoff for a fast and fixed execution time. On some applications it might actually be an advantage.

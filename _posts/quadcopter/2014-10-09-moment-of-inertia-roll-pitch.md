---
layout: post
title: Moment of inertia (roll/pitch)
date: '2014-10-09T22:35:00+08:00'
tags:
- moment of inertia
- bifilar pendulum
- measure
- experiment
- quadcopter
---
<iframe width="560" height="315" src="https://www.youtube.com/embed/XPmZ9xbgVds" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

**Moment of inertia with respect to the roll/pitch axes**

We're doing the same experiment as [this guy](https://www.youtube.com/watch?v=m9iHEanmNWc). The moment of inertia was measured with respect to the pitch axis, but we would expect the same result with respect to the roll axis because of the quadcopter symmetries. Conveniently the roll, pitch and yaw axes are principal axes of rotation.

Oscillation period of the bifilar pendulum:

- 20T = 15.867 - 2.555, T = 0.6656s
- 20T = 29.010 - 15.867, T = 0.65715s
- 20T = 42.154 - 29.010, T = 0.6572s
- 20T = 55.231 - 42.154, T = 0.65385s
- Average T = 0.65845s

Other experimental quantities:

- m = 0.958kg
- b = 0.235m
- L = 0.51m

Moment of inertia:

$$I = \frac{m g T^2 b^2}{4 \pi^2 L}$$

$$I = 0.0112 \:kg\:m^2$$

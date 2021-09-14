---
layout: post
title: no title
date: '2014-10-09T22:35:00+08:00'
tags:
- moment of inertia
- bifilar pendulum
- measure
- experiment
- quadcopter
tumblr_url: https://robokitchen.tumblr.com/post/99593827730/moment-of-inertia-with-respect-to-the-roll-pitch
---
<video id="embed-613f4f5142710931276881" class="crt-video crt-skin-default" width="400" height="400" poster="https://64.media.tumblr.com/tumblr_nd73mpCttR1sjwnlx_frame1.jpg" preload="none" muted data-crt-video data-crt-options='{"autoheight":null,"duration":60,"hdUrl":false,"filmstrip":{"url":"https://33.media.tumblr.com/previews/tumblr_nd73mpCttR1sjwnlx_filmstrip.jpg","width":"200","height":"200"}}' crossorigin="anonymous">
    <source src="https://va.media.tumblr.com/tumblr_nd73mpCttR1sjwnlx_480.mp4" type="video/mp4">
</source></video>  

**Moment of inertia with respect to the roll/pitch axes**

We’re doing the same experiment as [this guy](https://www.youtube.com/watch?v=m9iHEanmNWc). The moment of inertia was measured with respect to the pitch axis, but we would expect the same result with respect to the roll axis because of the quadcopter symmetries. Conveniently the roll, pitch and yaw axes are principal axes of rotation.

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


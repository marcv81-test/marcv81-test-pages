---
layout: post
title: no title
date: '2013-12-15T15:59:00+08:00'
tags:
- imu
- 2dof
- gimbal
- servo
- closedloop
tumblr_url: https://robokitchen.tumblr.com/post/70088066125/closed-loop-gimbal-control
---
<video id="embed-613f4f514fb3e979975378" class="crt-video crt-skin-default" width="400" height="225" poster="https://64.media.tumblr.com/tumblr_mxutu3XBj71sjwnlx_frame1.jpg" preload="none" muted data-crt-video data-crt-options='{"autoheight":null,"duration":14,"hdUrl":false,"filmstrip":{"url":"https://25.media.tumblr.com/previews/tumblr_mxutu3XBj71sjwnlx_filmstrip.jpg","width":"200","height":"112"}}' crossorigin="anonymous">
    <source src="https://va.media.tumblr.com/tumblr_mxutu3XBj71sjwnlx.mp4" type="video/mp4">
</source></video>  

Closed loop servo gimbal control. The IMU is fixed to the gimbal. A Proportional-Integral-Differential (PID) controller adjusts the servo position to compensate for the roll and pitch error. I first moved slowly to test the precision, then fast to test the response speed.


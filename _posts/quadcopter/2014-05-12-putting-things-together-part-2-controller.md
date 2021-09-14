---
layout: post
title: 'Putting things together - Part 2: Controller'
date: '2014-05-12T17:11:00+08:00'
tags:
- quadcopter
- controller
- pid
- tuning
- rate
- attitude
- motor
- mix
---
From previous experiments and code we can achieve the following.

- [Control the speed of each motor](https://robokitchen.tumblr.com/post/84759084465/490hz-esc-control)
- [Read the orientation accurately while the motors are spinning](https://robokitchen.tumblr.com/post/83673830666/putting-things-together-part-1-imu)
- [Read the transmitter sticks](https://robokitchen.tumblr.com/post/72492489551/decoding-a-ppm-sum-signal)

What we need next is to implement controllers which ensure the motors make the orientation follow the sticks. Just like in the previous [servo gimbal sketches](https://robokitchen.tumblr.com/post/70087874691/servo-gimbal-control) we'll use PID controllers.

Before anything else we [rewrote the PID class](https://github.com/marcv81/quadcopter/commit/f435ba1c162bf051f12005277b851002b1ee06f4) using the velocity algorithm. We also removed the output bounds and included the sampling time in the formulas. The velocity algorithm is a convenient implementation which does not change anything in the PID behaviour.

**Transmitter sticks**

The sticks values range from -100 to 100. We programmed the controller so that a switch kills the throttle.

- When the throttle stick is bellow a certain threshold (-80) we kill the motors altogether. Good for safety.
- When the sticks are within a certain range of the center (+/-5), we set them to zero. This shall help to dissociate software and control issues from sticks centering or trim issues.

**Motor mix**

We are using a +4 configuration and [already decided](https://robokitchen.tumblr.com/post/78482282412/flight-controller) of the following.

- Front: CW
- Back: CW
- Left: CCW
- Right: CCW

This means that the motor mix shall be as follows.

- Front = Idle + Throttle + Pitch - Yaw
- Back = Idle + Throttle - Pitch - Yaw
- Left = Idle + Throttle + Roll + Yaw
- Right = Idle + Throttle - Roll + Yaw

The potential problem with the above formulas is that we might attempt to drive some motors with values which are out of bounds: we might be asking the motors to spin faster or slower than they possibly can. This would cause the PIDs to wind up. In order to remedy this issue, we can search for out of bounds control values and add the same constant to all the control values to achieve the following.

- Keep all the control values within bounds.
- Maintain the difference between each of the control values, so that the roll and pitch can still be controlled.

Idle should be adjusted so that the quadcopter does not gain or loose much altitude when the throttle is centered. This is only a rough adjustment.

Because our ESCs run the SimonK firmware we don't necessarily need to drive them with high level signals between 1ms and 2ms. Instead we configured them to handle high level signals between 0.2 and 2ms. This almost doubles the ESCs control resolution. Idle was set to 1ms.

**Roll and pitch rates (rate mode)**

The most simple mode to implement is the [rate mode](https://github.com/marcv81/quadcopter/commit/b33cf043b2c671b4113e2edc304edb97fdb98be7). Because a +4 quadcopter is symmetric we can tune the roll and pitch rates controllers at the same time. These controllers shall ensure the rotation rates on the roll and pitch axes are proportional to the ailerons and elevator sticks.

We tuned these controllers as PDs. I held the quadcopter above me, with a hand at the extremity of two opposed booms, and I controlled the transmitter with my feet. A proper test rig would have been more appropriate.

First we set $K\_d$ to zero and increase $K\_p$ progressively until we get light oscillations. Then we increase $K\_d$ progressively until the oscillations dampen, but we stop before the quadcopter gets twitchy. We obtained $K\_p = 150$ and $K\_d = 8$.

**Yaw rate**

The yaw rate controller ensures the rotation rate on the yaw axis is proportional to the rudder stick.

We tuned this controller as a simple P. Because the motors torque control the yaw +4 quasdcopters aren't too responsive on this axis, so using only P shall be good enough. I simply put the quadcopter on the floor and tried to yaw.

We increased $K\_p$ progressively until the response felt acceptable. We couldn't reach oscillations. We obtained $K\_p = 500$.

**Roll and pitch angles (attitude mode)**

The [attitude mode](https://github.com/marcv81/quadcopter/commit/592156a8373c5254e0757b7ae110d22cd292f824) is more elaborate than the rate mode. Instead of controlling the roll and pitch rotation rates, the ailerons and elevator sticks control the roll and pitch angles. We reused all the controllers from the rate mode. However we drove the roll and pitch rate controllers with new roll and pitch angles controllers instead of the sticks.

We tuned these new controllers as PIs. They were tuned while flying.

First we set $K\_i$ to zero and increase $K\_p$ progressively until we get light oscillations. We then reduce $K\_p$ until the quadcopter is stable again. We now increase $K\_i$ until we get light oscillations, and reduce $K\_i$ until the quadcopter is stable again. We got $K\_p = 5$, $K\_i = 20$.

**Performance**

The quadcopter performs well indoors. Outdoors it may be unstable when there are sudden wind gusts or when there are rapid stick changes from one extreme angle to the other. The quadcopter is not useable, but it can stay airborne reliably enough to allow characterisation and further tune the controllers.

---
layout: post
title: Lift force measurement
date: '2014-09-15T00:00:00+08:00'
tags:
- torque
- force
- rpm
- scale
- weight
- gravity
- simonk
- linear
- quadratic
- model
---
Let's measure how much lift force each motor/propeller is generating.

**Measure**  
  
The idea is to put the quadcopter on a scale and observe how much less weight is measured when a motor spins. To reduce the disturbances this should be done indoors and with enough clearance to avoid any ground effect. As all the motors/propellers should produce similar results I decided to measure the front one only.  
  
I [modified the QuadMotorTest sketch](https://github.com/marcv81/quadcopter/commit/71c3a7c3d8ce4d86e915c37da0a31218179a8c6f) to help us with the measure.

- The PWM high-level time is now printed on the serial port.
- The PWM is now adjustable on the AUX channel which I set to a knob on the transmitter.
- The elevator and ailerons now activate the motors independently, and the throttle and rudder activate the pairs of opposite motors.

It is now easy to control exactly the input of each ESC/motor. It will help us relate the PWM to the generated lift force.

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2014-09-15-lift-force-measurement-1.jpg)
{:refdef}

I spent £12 on a small kitchen scale which can weigh objects up to 3kg. I taped 2kg worth of books to the quadcopter but I could only test the 200-900ms range before it would fall from the scale. ASDA makes a [fantastic £8 kitchen scale](https://direct.asda.com/George-Home-Electronic-Flat-Scales-10kg/001468048,default,pd.html) which can weigh up to 10kg. I taped the quadcopter to a 8kg box of books and I could measure the whole 200-2000ms range without any problem.

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2014-09-15-lift-force-measurement-2.jpg)
{:refdef}

In order to acquire as many samples as fast as possible, I setup a camera on a tripod to take photos of the scale and the computer screen as I increased the motor speed. I ran two data acquisition rounds: each produced more than 100 samples in 3-4 minutes. I noted the unloaded battery voltage at the end of the measure to differentiate the data sets: these are not the voltages under load while the motor was spinning.

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2014-09-15-lift-force-measurement-3.jpg)
{:refdef}

**Analysis**

[Apparently](https://andrew.gibiansky.com/blog/physics/quadcopter-dynamics/) the laws of physics predict that the force magnitude should be proportional to the square of the RPMs. The SimonK ESC firmware claims to have a linear throttle response, which I suppose is meant to be understood in terms of RPMs. If this claim (and physics) are true, then the square root of the lift force could be expressed as a linear function of the PWM. This turns out to be a reasonable estimate, especially if we exclude the extremely high and low PWM values.

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2014-09-15-lift-force-measurement-4.jpg)
{:refdef}

f(x) = 0.00118415*x + 0.30756716

The linear RPMs estimate leads to the following quadratic force estimate. Again it's not too bad, especially if we exclude the extremely high PWM values.

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2014-09-15-lift-force-measurement-5.jpg)
{:refdef}

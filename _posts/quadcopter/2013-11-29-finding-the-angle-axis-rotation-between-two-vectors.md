---
layout: post
title: Finding the angle-axis rotation between two vectors
date: '2013-11-29T19:53:00+08:00'
tags:
- math
- vector
- rotation
- axisangle
---
In this post we explain a method to find the [angle-axis](https://en.wikipedia.org/wiki/Axis%E2%80%93angle_representation) representation of the "shortest arc" rotation which transforms $$\vec{u}$$ into $$\vec{v}$$. The vectors are non-null, non-collinear, and the coordinate system is [right-handed](https://en.wikipedia.org/wiki/Right-hand_rule#Right-handed_and_left-handed_coordinates).

The method is simple enough but I'm writing it here because I couldn't find it anywhere else.

Let's call $$\theta$$ the angle between $$\vec{u}$$ and $$\vec{v}$$. By definition $$0 < \theta < \pi$$.

We are looking for the vector $$\vec{a}$$ orthogonal to the plane containing $$\vec{u}$$ and $$\vec{v}$$, which has a norm of $$\theta$$, and which points in the direction such that the rotation from $$\vec{u}$$ to $$\vec{v}$$ follows the [right-hand rule](https://en.wikipedia.org/wiki/Right-hand_rule#Direction_associated_with_a_rotation).

Let's define:

$$\vec{t} = \frac{\vec{u} \times \vec{v}}{|\vec{u}| |\vec{v}|}\tag{1}$$

The cross product is orthogonal to the plane defined by the vectors it is calculated from, so $$\vec{t}$$ and $$\vec{a}$$ are collinear.

The cross product points in the direction determined by the right-hand rule, so $$\vec{t}$$ and $$\vec{a}$$ point in the same direction.

We can hence write:

$$\vec{a} = \alpha \vec{t} \text{, with } \alpha > 0\tag{2}$$

Where:

$$\alpha = \frac{|\vec{a}|}{|\vec{t}|}$$

$$\alpha = \frac{\theta}{|\vec{t}|}\tag{3}$$

By definition of the cross product:

$$|\vec{u} \times \vec{v}| = \sin{\theta} |\vec{u}| |\vec{v}|\tag{4}$$

From (1) and (4) we obtain:

$$|\vec{t}| = \sin{\theta}\tag{5}$$

$$\theta = \arcsin{|\vec{t}|}\tag{6}$$

It is worth noting that because $$0 < \theta < \pi$$ we are not loosing any information using arcsin.

From (3) and (6) we get:

$$\alpha = \frac{\arcsin{|\vec{t}|}}{|\vec{t}|}\tag{7}$$

From (3) and (5) we get:

$$\alpha = \frac{\theta}{\sin{\theta}}\tag{8}$$

Finally with (1), (2), and (7) we reach the conclusion **(A)** :

$$\vec{a} = \frac{\arcsin{|\vec{t}|}}{|\vec{t}|} \vec{t} \text{, with } \vec{t} = \frac{\vec{u} \times \vec{v}}{|\vec{u}| |\vec{v}|}$$

Also, when $$\theta$$ is small we can use the small-angle approximation $$\theta \approx \sin{\theta}$$.

From (8) we get:

$$\alpha \approx 1\tag{9}$$

When the angle between $$\vec{u}$$ and $$\vec{v}$$ is small, with (1), (2), and (9) we reach the conclusion **(B)**:

$$\vec{a} \approx \frac{\vec{u} \times \vec{v}}{|\vec{u}| |\vec{v}|}$$

Implementation notes:

- If any of the vectors are null we risk a division by zero. We shall check for this case.
- If the vectors are collinear we risk a division by zero unless we are using the small-angle approximation. We shall check for this case.
- If the vectors are collinear the small-angle approximation gives a correct rotation if $$\theta = 0$$, but an incorrect rotation if $$\theta = \pi$$.

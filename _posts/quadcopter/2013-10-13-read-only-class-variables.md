---
layout: post
title: Read-only class variables
date: '2013-10-13T23:50:00+08:00'
tags:
- arduino
- code
tumblr_url: https://robokitchen.tumblr.com/post/63965310728/read-only-class-variables
---
We want to provide read-only access to variables containing the computed value of the accelerometer, the gyroscope, the magnetometer, the barometer, etc. I could think of three ways to do this:

1. Declare the data public anyway. The user may overwrite the data.
2. Declare the data private, and provide a public function to retrieve its value. Hopefully the compiler can inline the function and we won’t waste any code size or loose any performance.
3. Declare the data private, and provide a public constant reference to it. We shall expect to waste a one-off address (4 bytes) for the reference, and also waste a tiny bit of code size and loose some performance for each call.

I created a [test sketch](https://github.com/marcv81/robokitchen/blob/9c570266aae4dd733a1f3b401837bd41df76c613/sketches/TestReadOnly/TestReadOnly.ino) to compare these three implementations. Each implementation was tested to access two different types: an int, and a struct containing three ints. Each implementation was also tested with 1, 4, and 8 calls to retrieve the value. This was to attempt to isolate the one-off size cost and the per-call size cost. The inline accessor and const reference implementations were compared against the public variable implementation for increased code size. Performance was not monitored.

<figure class="tmblr-full" data-orig-height="150" data-orig-width="500"><img alt="image" src="https://64.media.tumblr.com/39b7f96d328c2b72439890044397644f/031083498cec1f2d-42/s540x810/6830a4156eb253a6f94eed9453b396f92a417317.jpg" data-orig-height="150" data-orig-width="500"></figure>

**Findings:**

When the accessed value is an int, the accessor call didn’t generate any code overhead compared to accessing the public variable directly. The compiler most probably inlined the function as expected. When the accessed value is a struct, the accessor generated a very significant overhead.

When using a const reference, there is a small overhead which is independent of whether the value is an int or a struct. This was to be expected because references behave the same way regardless of the object they point to.

In the cases where the code size increased, it was hard to isolate the one-off size cost and the per-call size cost. There is no apparent proportional relationship between the increased code size and the number of calls. This is probably because the compiler made some other optimisations.

**Conclusion:**

The best solution is to use public accessors and make sure they only give access to base types. There is no overhead and the values are adequately protected from modifications.

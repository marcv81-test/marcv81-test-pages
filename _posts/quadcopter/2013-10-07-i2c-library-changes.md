---
layout: post
title: I2C improvements
date: '2013-10-07T22:21:00+08:00'
tags:
- arduino
- i2c
tumblr_url: https://robokitchen.tumblr.com/post/63404214442/i2c-library-changes
---
Before continuing we made quick improvements to the I2C library to reduce the code footprint:

- Saved [30 bytes](https://github.com/marcv81/robokitchen/commit/3b98c0a9beb32c987bff1b6c3b4eea881cdf7131) removing unwanted functions and their associated variables.
- Saved another [30 bytes + 4 bytes per function call](https://github.com/marcv81/robokitchen/commit/4e99e9c24c0d9fb758081350936ec519355bd60c) making the functions static and removing the constructor.
- Saved yet another [130 bytes](https://github.com/marcv81/robokitchen/commit/309b63e254cfa45365e0db872076c9a36c4075dc) defining the I2C speed and timeout as constants.

These may not be significant savings but they didn’t require much effort and we didn’t loose any useful feature.

We also made more general improvements:

- [Defined constants](https://github.com/marcv81/robokitchen/commit/5a6c0f4011784d1f84fb062f4cf5093c67cadc01) for the I2C timeout error codes (which were implemented as magic numbers).
- Use [microseconds rather than milliseconds](https://github.com/marcv81/robokitchen/commit/ebbf159b1b91885fe5d1bd96af0363e7f791f35d) for the I2C timeout.
- Various code trimming and formatting improvements.

This makes the I2C library more readable and more useful for our purposes.


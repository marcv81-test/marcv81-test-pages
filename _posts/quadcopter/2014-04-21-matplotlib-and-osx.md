---
layout: post
title: Matplotlib on OSX
date: '2014-04-21T23:35:00+08:00'
tags:
- matplotlib
- osx
---
Matplotlib was easy to install on Ubuntu.

_sudo apt-get install python-matplotlib_

The same cannot be said of OSX. Matplotlib 1.3.1 for Python 2.7 suffers from a numpy version mismatch when used with the latest Python 2.7.6. I had to install Matplotlib 1.2.1 instead, which by the way requires XQuartz.

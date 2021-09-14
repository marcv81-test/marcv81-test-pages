---
layout: post
title: Sharing code between sketches
date: '2013-10-24T01:13:00+08:00'
tags:
- arduino
- code
- osx
---
My strategy for every I2C sensor so far has been the following:

1. Write the sensor-specific code in an [Arduino library](https://arduino.cc/en/Hacking/LibraryTutorial)
2. Write a test sketch that uses this library

The idea was that I could then pick any combination of sensors, create a new sketch, include the right libraries, and start writing code to process the acquired data.

I didn't write libraries by choice though. Arduino does not provide any other way to share code between sketches. Unless it is in a library, all the code used in a sketch must be in a single folder, and it is not possible to have more than one sketch in that folder. I blame the [build process](https://arduino.cc/en/Hacking/BuildProcess).

The libraries approach has two caveats.

1. Every time I modify a library I need to copy it to ~/Documents/Arduino/libraries. In other environments I can automate the libraries deployment, not with Arduino.
2. The preprocessor macros defined in the sketches are not available in the libraries. It is not possible to configure libraries from sketches at compile time.

Eventually I figured I could use the following workaround.

1. I created [symbolic links to the required shared files in my sketch folder](https://github.com/marcv81/quadcopter/commit/10bb0c59a40d022e19ee9e27a19d6dcb0b697b9c). The only required change for this to work was to include the shared code using double quotes instead of angle brackets. No more copying files after this.
2. Now that our shared code isn't technically in an Arduino library, it can share preprocessor directives with the rest of the sketch. I included a [config.h in each shared file](https://github.com/marcv81/quadcopter/commit/736ed31f1c8209b8c6aa689aaf00b15091fd8f10), which is for each sketch to implement. Shared configuration goes there.

Sharing code between sketches is now much more convenient.

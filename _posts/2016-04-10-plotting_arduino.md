---
comments: true
layout: post
title: Plotting Data From Arduino
tags: Arduino serial
---

# Plotting Data from Arduino

Debugging Arduino code is hard without support like J-tag. Most of the time we rely on serial logs.
Since Arduino applications are loops in nature, the data naturally manifest itself with time-series properties.
Charting and plotting data from Arduino would make a lot of debugging easier, compared to looking at the raw data from the serial monitor window.

So I made a tool for plotting data from the serial port.

All of the tools and libs are ready-made, connecting the dots was great fun.

https://github.com/DFRobot/DFSerialChart

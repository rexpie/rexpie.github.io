---
comments: true
layout: post
title: Controlling PWM with Your Phone
tags: Arduino firmata socket.io Johnny-five HTML5 IOT
---

# Controlling PWM with Phone

Motion based projects are fun. There are a few gyro and accelerometer that works with Arduino, but
they are not free. However most phones comes with those sensors and we are going to take advantage
of that - making a remotely controlled PWM pin with your phone.

[![image]({{ site.url }}/assets/remotePWM.png )](https://youtu.be/R54YdTUsqu8)

# System Environments

Hardware list:

* Mac OS 10.11.5
* Arduino Uno board of your choice, I am using one from [DFRobot](http://www.dfrobot.com/index.php?route=product/product&product_id=838&search=uno&description=true)
* LED or motor

# Software setup

## Flash Firmata into Your Uno Board

There is a nice [tutorial](http://www.instructables.com/id/Arduino-Installing-Standard-Firmata/) for you.



## Download & Install node.js [optional]

Please [nodejs.org](http://nodejs.org) for instructions.


## Clone or download example project

Please follow the [github example project](https://github.com/rexpie/remotePWM).


## Wire the PWM device

In the example project we are using Pin 11.  
You can change the pin number as long as
it is still a PWM pin. Remember to match the code and your wiring if you decide to modify the pin number.


## Build and Run

    npm install && npm start

There is a [video](https://youtu.be/R54YdTUsqu8) showing the result.

# How It Works

## Server side

1. The server script starts a tiny web server to host the html and scripts.
2. It also generates a QR code for you to scan to link created.
3. Then it starts the Johnny-five instance.
4. Finally it starts a socket.io server to listen to phone events.


## Mobile side

1. Once the QR code is scanned, the QR code app ( or you can use whatsapp ) opens the browser
2. HTML5 code starts to sense phone orientation events and forward that information to our socket.io server

## Interaction

1. Server gets orientation info from mobile device, translate raw data and map to PWM output
2. Johnny-five instance channels that info into the board
3. Firmata firmware on the board decodes the info and outputs corresponding PWM wave on the pin

Voilà.

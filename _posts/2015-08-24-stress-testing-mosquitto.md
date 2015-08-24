---
layout: post
title:  "Stress testing Mosquitto MQTT Broker"
date:   2015-08-23 10:55:00
categories: mqtt
---

# What am I testing for

I am testing the server under a very **specific** scenario. Very large number of subscribers and publishers, but very little traffic for each one. I am trying to build a sensor network server with push capabilities.

* It is written in C, with tiny memory footprints for each connection, unlike Apache's ActiveMQ or Apollo which creates a thread object for every live connection.
* It is single threaded
* It is easy to install, simple to configure and quick to run

I am not saying other brokers are not superior on those aspects. Simplicity is what I am after and after all, I think that managing a network of broker cluster exceeds the boundary of software design and need system architecture behind it. So I want to start with simple tests on a single server and see how it goes.

Here is an [article](http://w3yyb.sinaapp.com/archives/1601) ~~in Chinese~~ of an emperical study on MQTT broker performances. Basic translation of the conclusion is: 

* Downsides: Mosquitto only utilizes one CPU; EMQTT writes too much disk io and has low throughput; ActiveMQ/Apollo eats a lot of RAM and sometimes crashes.
* Upsides: EMQTT & Mosquitto are quite stable. EMQTT has high cuncurrent connection counts. Apache ones can use multicore.


# Prerequisites

I am testing on two Linux/Unix like systems (actually they are 2 virtual Ubuntu desktop hosted in a VMWare env, if you must know) connected in the same LAN. Let's call them *Alice* and *Bob*.

# Setting up the environment

Here are the steps to execute the stress test. I can give you the results beforehand:

> It can handle at least 20k simultanious connections at a speed of 7000+ messages per second for a single 2.1g core virtual server with just 12MB of mem.


## Changing System File Limitations

*** Do this on both Alice and Bob ***

Your system may restrice open files per process and system-wide file handle counts.

I found a [tutorial](https://rtcamp.com/tutorials/linux/increase-open-files-limit/) for linux/Unix like systems.

Basically you edit `/etc/security/limits.conf` and put in:

`*         hard    nofile      500000`

`*         soft    nofile      500000`

You may also need to modify `/etc/sysctl.conf` and put in:

`fs.file-max = 2097152` and run `sysctl -p`

*sudo if necessary*



## Mosquitto Server

*** Do this on Alice ***

If you want to use the latest version, go to their official download [link](http://mosquitto.org/download/).

Or if you want to quickly install a curated version for you system, you can also check out their recipes in the same page.

I built the server from its source. I found out that I need:

* g++
* uuid-dev
* libc-ares-dev
* libssl-dev

Then you can execute `make && sudo make install` and check if you have `mosquitto` available after that.

## Configuring and Starting Mosquitto

*** Do this on Alice ***

You can refer the [documentation](http://mosquitto.org/man/mosquitto-conf-5.html) for the detailed parameters, but I focus on the `max_connections` configuration value and change it to `-1`(unlimited)

Then just execute `mosquitto`. You can add the `-d` switch to run it in the background. I just like to see the logs.



## Benchmark tool

*** Do this on Bob ***

I used a pretty cool tool called [mqtt-bench](https://github.com/takanorig/mqtt-bench)

I can't get it to work using the `go get` command in its documentation, so I downloaded the [binary](https://github.com/takanorig/mqtt-bench/wiki/Download) as it advised.

## Starting Test

*** Do this on Bob ***

The CLI command for the benchmark tool looks like this:

`./mqtt-bench  -action=p -broker="tcp://192.168.0.89:1883" -clients=20000 -count=10`


## Checking Status on Alice

*** Of course do this on Alice ***

You can use the `top` command to see the CPU and memory usage of Alice.
For mine, I see a single core surge to 100 with a 0.3% memory ( out of 4G, so 12M ) when the connections are being accepted.

You can also check the network connections using:

`netstat -pant | grep mosquitto | wc -l`


## Conclusion

I already gave the results. 

`//TODO` Next time maybe I can use some more Bobs to connect to Alice and see how much fun they can have.

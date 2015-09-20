---
comments: true
comments: true
comments: true
layout: post
title:  "Stress testing Mosquitto MQTT Broker Update 1"
date:   2015-08-31
tags: mqtt mosquitto
---
comments: true
comments: true
comments: true

# Stress Testing Mosquitto Update 1

Last time I left a bit in the `TODO`, saying that I will test with multiple clients on a host. I did.

## Env setup

To compare the testing, I will give you the hardware settings:

Host			|CPU Core#	| Mem	
---------------	|----------	|-----
comments: true
comments: true
comments: true
VMware Guest	|1			|4g
Physical		|2			|8g

The clients are all VMware Guest Ubuntu Desktop machines cloned from the server. They have the same file limits, etc, nothing special.

I use bridge, not NAT for the network connectivity for the VMWare Guests, so they are just like other machines setup on the local network, not wrapped up inside the VM.

## Results

Quite interesting. The VM Guest, no matter how I tune the numbers, cannot establish more than 655XX TCP connections, regardless of the IP/port combinations.

I tried to use HAProxy with another guest machine, and I get ~32000 frontend connections, and ~16000 backend connections to each of my two backend Mosquitto servers. So that is about 655XX TCP connection in total.

I kind of scratch my head when I cannot pass that threshold. I checked the values everywhere and I am sure my configuration is able to host more TCP connections than 655XX. Besides, this number is way too suspicious. I tried half a dozen times and the cap is always 655XX. Then I suspected VMWare.

Okay I need proof.

I used the same step to install a vanilla Ubuntu with the same iso image, followed the same steps I wrote myself to setup the server on the physical machine and guess what, I got more connections as expected.

### Final stats

Host			|CPU%	|RAM%	|Conn Speed		|Max Conn
---------------	|------	|------	|---------------|--------
comments: true
comments: true
comments: true
VMware Guest	|90%+	|0.4%	|2000+/s -> ~100/s| **65,535**
Physical		|90%+	|0.4%	|3000+/s -> ~100/s| **100,000+**

The speed of establishing new connections wears down gradually. Mosquitto can build up 10K connections in a few seconds, but the speed slows down after that.

I got over 100K live TCP connections with a physical machine, using only 0.4% memory. That is very impressive indeed.
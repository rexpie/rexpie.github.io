---
layout: post
title:  "Mosquitto Troubleshooting"
tags: mqtt, mosquitto
---

# Issues with mosquitto_pub

When I tried to execute the `mosquitto_pub` command, it complains about a lib missing:

**`rex@ubuntu:~$ mosquitto_pub -t '/mqtt-bench/benchmark' -m 'testContent'`
**

`mosquitto_pub: error while loading shared libraries: libmosquitto.so.1: cannot open shared object file: No such file or directory`

I **built** this from source. Well seems like mosquitto needs a bit more consistent at housekeeping when it tries to install and execute.

Let's dig in.

First use strace to look what it is looking for:

**`rex@ubuntu:~$ strace mosquitto_pub -t '/mqtt-bench/benchmark' -m 'testContent'`
**

`open("/usr/lib/x86_64/libmosquitto.so.1", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
`

`stat("/usr/lib/x86_64", 0x7fff1e3b13f0) = -1 ENOENT (No such file or directory)
`

`open("/usr/lib/libmosquitto.so.1", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
`

Okay so it looks through `/usr/lib`.

Where **is** the lib then?

**`rex@ubuntu:~$ locate libmosquitto.so|xargs ls -l`
**
`/usr/local/lib/libmosquitto.so
/usr/local/lib/libmosquitto.so.1
`

So it looks at `/usr/lib` rather than `/usr/local/lib`. Fair enough, I did not specify the prefix when building.

Fix is easy:

**`sudo ln -s /usr/local/lib/libmosquitto.so.1 /usr/lib/libmosquitto.so.1`**


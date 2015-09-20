---
comments: true
comments: true
comments: true
layout: post
title:  "Scratch with Johnny-five"
tags: Visual Programming
---
comments: true
comments: true
comments: true

# Scratch Extension with Johnny-five

Previous [article](http://rexpie.github.io/2015/08/23/extending-scratch.html) illustrated the architecture of the Scratch extension.

We can replace the PyMata part with anything that can act as a broker between Firmata and the HTTP.

Nodejs has a rich library and [Johnny-five](http://johnny-five.io)(J5) is one of the most popular Firmata clients. It is actively maintained by [Rick Waldron](https://github.com/rwaldron) as of this document is written. 

I was amazed by how Rick handles issues with chips. I told him one of the LCD chip is not working well with j5 [link](https://github.com/rwaldron/johnny-five/issues/326#issuecomment-135857146). He said the would buy that chip online and test it after. He did things like this multiple times on other occasions as well. Hats off.

# Let's do this

I want to build a version that works on Windows to start. Both x64 and x86 OS should be able to execute the binaries. 

Why does this matter? Isn't Nodejs cross platform?

Because we need to talk through the serial port. J5 depends on [serialport](https://www.npmjs.com/package/serialport) package and usually it need compiling. The documents of `serialport` package has detailed instructions as how to deal with 'build fail' issues.

## Dependencies

* Nodejs x86 version for compatibility 
* Python 2.* matching OS bitness ( 32 for 32bit OS, 64 for 64bit OS)

After you get your nodejs and python ready, remember to put the bin path into your PATH variable. Then you create a project path for the helper app. Let's call it 5jhelper.

	cd j5helper
	npm install johnny-five
    npm install serialport

Sometimes serialport will install successfully with johnny-five. If serialport failed to install, best solution would be RTFM and install it yourself. It says that it already has the binaries shipped with the package for most common use cases. I am not sure how it determines the binary version, but I have to build myself from time to time.

> You ***MUST*** install the serialport package for this to work


## Building Helper App with J5

You can go to the example folder in the j5 package and have some fun with whatever you have on your hand like Arduino and LED. Or you can checkout the [videos](https://www.youtube.com/results?search_query=nodebots) as well.



### HTTP Server

You can use the http module and make it listen on any port you have defined in your extension descriptor, by default the port id 12345.

You need to remember to call `response.end()` promptly or Scratch will hold on to that HTTP request till it times out. The Scratch source code, at this time, is just a while loop firing HTTP requests on a single thread, so it blocks.

*Remember to send data back to Scratch as quickly as possible.*

You can parse the HTTP requests sent from Scratch easily as they are in a very simple [slash-delimited format](https://github.com/LLK/scratchx/wiki).


### J5 backend

#### Caching your data
Usually j5 API returns very fast, you do not need to worry about the timings. But when you are dealing with loops it becomes somewhat more unpredictable, because writing to the serial device takes time, processing Firmata protocol on Arduino takes time and reflecting on the actual device also takes time. It becomes even slower when you are using a [USB bluetooth link](http://www.dfrobot.com/index.php?route=product/product&product_id=1220&search=ble&description=true#.VeFMybR7wW8) to wirelessly connect to your board.

I tried to write messages to a LCD. The loop keeps the LCD blinking and flashing and rewriting. Every time a message arrives I call `lcd.clear()` then `lcd.print(msg)`.

> Keep the output values cached, only write when the value changes.


# Disclaimer

With the movement of ScratchX( now Beta) This document might be obsolete soon.





---
comments: true
layout: post
title:  "Fruit Piano with Firmata"
tags: Visual Programming
---


In the previous post I said it would be more expensive to implement a Firmata based approach. I was wrong. Firmata SYSEX command set is quite easy to extend actually.

# Extending Firmata on Arduino

I was strolling on google when a wild [tutorial](http://blog.s4a.cat/2015/03/13/Extending-Firmata-for-Snap4Arduino.html) appears. Thanks to this piece of example which encouraged me to give Firmata a try.

I was previously looking at Johnny-five's code and was properly scared off. The complexity is just too damn high.

However we are not toying with Johnny-five this time, just plain old Firmata.

First we open up the StandardFirmata ino file and insert this code:

{% highlight C++ %}
    case 0x08:
      Firmata.write(START_SYSEX);
      Firmata.write(0x08);
      Firmata.write(argv[0]);
      Firmata.write(readCapacitivePin(argv[0]));
      Firmata.write(END_SYSEX);
      break;
{% endhighlight %}

Also insert the readCapacitivePin function to the end of the file. You can find the code in [this post](http://playground.arduino.cc/Code/CapacitiveSensor).

# Extending Firmata Nodejs client

You can get your firmata module from npm.

Once you have installed the module, go to its directory and find `lib/firmata.js`.
Add the following code:

{% highlight js %}
var PIN_CAPACITANCE = 0x08;

SYSEX_RESPONSE[PIN_CAPACITANCE] = function(board){
  var pin = board.currentBuffer[2];
  var cap = board.currentBuffer[3];
 // console.log(board.currentBuffer.toString("hex"));
  if (board.pins[pin]){
    board.pins[pin].cap = cap;
  }
}

Board.prototype.queryPinCapacitance = function(pin) {
  this.transport.write(new Buffer([START_SYSEX, PIN_CAPACITANCE, pin, END_SYSEX]));
};
{% endhighlight %}


# Testing

{% highlight js %}

var serialPort = require("serialport");
var defaultPort;

var firmata = require('./lib/firmata.js'),
    repl = require('repl');


serialPort.list(function(err, ports) {
    ports.forEach(function(port) {
        // Arduino COM ports usually have the manufacturer name start with "Arduino"
        if (port.manufacturer.toLowerCase().indexOf("arduino") > -1) {
            defaultPort = port.comName;
		    var board = new firmata.Board(defaultPort, function() {
		        console.log('Successfully Connected to ' + defaultPort);
		        repl.start('firmata>').context.board = board;
		    });
        }
    });
});
{% endhighlight %}

You can type in board.queryPinCapacitance(13) and try it out. Board setup is easy:

![image](http://playground.arduino.cc/uploads/Code/capacitive.jpg)

Image curtesy to Arduino


# Firmataized Fruit Piano

Here is your Helper App code

{% highlight js %}
var serialPort = require("serialport");
var firmata = require("firmata");
var http = require("http");

var defaultPort;
// holding the values of each port in a map, updated continuously
var fruits = {};

serialPort.list(function(err, ports) {
    ports.forEach(function(port) {
        // Arduino COM ports usually have the manufacturer name start with "Arduino"
        if (port.manufacturer.toLowerCase().indexOf("arduino") > -1) {
            defaultPort = port.comName;
            var board = new firmata.Board(defaultPort, function() {
                console.log("Connected to:" + defaultPort);
                setInterval(function() {
                    for (var pinNum = 6; pinNum < 14; pinNum++) {
                        board.queryPinCapacitance(pinNum);
                        fruits[pinNum] = board.pins[pinNum].cap;
                    }
                }, 1);
            });
        }

    });
});

// global http server
var server;
server = http.createServer(function(request, response) {

    // start recording request entry time

    if (request.url != '/poll') { // not poll
        // print non-polling command sent by Scratch		

    } else { // Polling

        Object.keys(fruits).forEach(function(pinName) {
            response.write("capacity/" + pinName + " " + fruits[pinName] + "\n");
        });

    } // end if

    response.end();
});
server.listen(23456);

{% endhighlight %}


# Ready to Go

You can just use the Scratch project in the previous Fruit Piano Scratch project. The Scratch project is athogonal to our Helper App implementation hence no change is needed.
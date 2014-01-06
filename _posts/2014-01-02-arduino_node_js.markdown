---
layout: post
title:  "Arduino via Node.js"
date:   2014-01-02 12:00:00
tags: software node arduino
comments: true
permalink: /software/node/arduino/2014/01/02/arduino_node_js.html
---

Get an [Arduino](http://arduino.cc) and a little self-imposed time on the bench, and you too can play around with blinking lights and reading sensors. I picked up the [SparkFun Inventor's Kit](https://www.sparkfun.com/products/11576) to try my hand with with a wide variety of electronic parts.  It includes a great little book diagramming the example circuits and code samples for each. 

I'm not afraid of writing code in C via the Arduino IDE (long time Java coder, with no fear of diving into other languages), but I can't say I'm terribly excited about it either.  I came across the [Johnny-five](https://github.com/rwaldron/johnny-five) library for Node.js and I was very interested.  The combination of rapid prototyping and (some-what) functional language creates a very interesting playground to try out these examples. It gives me the possiblility of both exploring Node.js and arduino.  

Johnny-five, in a nutshell, provides a API built on top of [Standard Firmata](http://firmata.org/) that abstracts the basics into a set of simple objects:  Board, Sensor, Led, etc.  These give the programmer a rather simple set of items with which to build complex control software for Arduino boards.  The limitation, of course, is that the device must be connected at all times (though, I do wonder if this is portable to the [Yun](http://arduino.cc/en/Main/ArduinoBoardYun), with its onboard linux computer). 

Blinking up an LED is easy:

``` javascript
var j5 = require("johnny-five");
var board = new j5.Board();
var LEDPIN = 9;

board.on("ready", function(){
  var led = new j5.Led(LEDPIN);

  led.strobe();
});
```

We can save the above code, plug an LED into our breadboard appropriately and run the code.  This will connected to the device, the led will begin to blink, and we'll be dropped into a repl.  The `led` can even be made available to the REPL, by adding:

``` javascript
board.repl.inject({
  led: led
});
```

Now we can run commands on the led as we see fit.  

```
>> led.off();
>> led.pulse();
```
and we've changed the led from strobing to pulsing. 

All was well and good until I moved on to the next example.  This involes reading the values off of a potentiometer, and then modifing the blink frequency of an LED.  Sounds pretty simple, and when we take a look at the [sample code](https://github.com/sparkfun/SIK-Guide-Code/blob/master/Circuit_02/Circuit_02.ino) from SparkFun, it looks simple, too.

``` cpp
sensorValue = analogRead(sensorPin);    
digitalWrite(ledPin, HIGH);     
delay(sensorValue);             // Pause for sensorValue
digitalWrite(ledPin, LOW);      
delay(sensorValue);             // Pause for sensorValue
```

Initially, this looks pretty simple using Johnny-five: 

``` js
var potentiometer = new j5.Sensor({
    pin: "A0",
    freq: 250,
    threshold: 5
});

var led = new j5.Led(13);
```

Now, the question is, read the values and change the loop interval?  It get's a bit more complicated, thanks to Node's asynchronousity:

``` js
potentiometer.on('data', function() {
  if(last_sensor_value === this.value){
    return;
  }

  console.log(this.value, this.raw);
  last_sensor_value = this.value;

  if(loop) {
    clearInterval(loop);
  }
    
  loop = setInterval(function() {
    led.on();

    setTimeout(function() {
      led.off();
    }, last_sensor_value);

  }, last_sensor_value * 2);
});
```

You'll note that we have set a `threshhold` value.  We get a bit of data 'wobble' - ±1 usually - from when reading the value of the sensor.  The threshold helps keep us from receiving data events too often.  We could probably crank this up a bit, and perhaps save ourselves from having to keep track of the previous value.

The looping and timeout seems a bit clumsy as well.  Here, we find ourselves setting an interval for a millisecond value based on twice sensor's current value, such that we are off for `last_sensor_value` and on for `last_sensor_value` amount of time.  With a little search I'm sure we can find a another library to take improve our experience.  

Even the odd loopiness, Johnny-five is off to a good start.  


---
layout: post
title:  "Blink: A Hello World for Embedded Software"
date:   2014-04-23 15:30:00
tags: software clojure RaspberryPI Arduino
comments: true
---
The blink example is the tried and true "Hello World" of the embedded micro-controller/computer world.  This simple example is great not only for verifying that one knows how to wire up a simple LED, but also for demonstrating to the user how to control said LED in software. I've put together an example using the [Firmata protocol](https://github.com/peterschwarz/clj-firmata-samples/blob/master/src/firmata_samples/sik.clj#L10-L24), but let's take a look at how to do it on the Raspberry PI. Thanks to the great work already done on the [PI4J](http://pi4j.com/) library, this is a relatively painless task.  

We start by just grabbing a few dependencies:

    [com.pi4j/pi4j-core "0.0.5"]

Now, we can import a few classes from PI4J:

    (import '[com.pi4j.io.gpio GpioController GpioFactory GpioPin GpioPinDigitalOutput PinState RaspiPin]))

Seems like quite a few classes, but they provide a pretty concise way to communicate with digital pins.  Let's grab pin 1:

    (def gpio (GpioFactory/getInstance))
    (def pin (.provisionDigitalOutputPin gpio RaspiPin/GPIO_01 "MyLed" PinState/LOW))

We can loop and blink the light:

    (dotimes [n 100]
        (.toggle pin)
        (Thread/sleep 1000))

Once we're finished, we want to be sure we release all our resources - PI4J's GpoiController ensures that a PIN can only be accessed one at a time - and shut everything down.

    (.low pin)
    (.unprovisionPin gpio (into-array GpioPin [pin]))
    (.shutdown gpio)

And that's it - it's really that simple. 

A complete example can be found [here](https://github.com/peterschwarz/hello-gpio), which includes a stop and startable blinker. 
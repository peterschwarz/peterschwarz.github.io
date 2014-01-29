---
layout: post
title:  "Serial Access via Clojure"
date:   2014-01-28 18:00:00
tags: software clojure arduino
comments: true
---

Having recently played around with [Node.js and arduino (via Standard Firmata)]({% post_url 2014-01-02-arduino_node_js %}), I had a hankering to try it in my current favorite langauge, [Clojure](http://clojure.org).  I thought it be great to play with an arduino in a pure functional language, trading nodebots for clojurebots.

There's one library out there for connecting to Standard Firmata devices over a serial connection: [Cloduino](https://github.com/nakkaya/clodiuno).  After working with [Johnny-Five](https://github.com/rwaldron/johnny-five) on Node, though, I was missing some of the automatic detection, as well as the abstractions built on top.  Additionally, Cloduino is built on top of the [RXTX](https://github.com/rxtx/rxtx).  RXTX involves a bit more installation work, and even took a bit of work to track down the source (I think the previous link is the most accuration).

Cloduino also has a lot of parts of the puzzle wired together into one library.  The layers aren't exactly nicely separated out.  The `cloduino.firmata` namespace couples both the firmata code and the serial communication in one place, for example. 

I thought this would be a good time to build a new library for my purpose.  Well, three libraries:  one for serial communication, one for Firmata interaction, and finally one for arduino-related abstrations (leds, motors, and the like).  This gives me the opportunity to understand both building and providing clojure libraries, as well as diving a bit more deeply into details of communicating with an arduino in this fashion. 

The first part was easiest: [clj-serial](https://github.com/peterschwarz/clj-serial).  It's mainly a fork of [serial-port](https://github.com/samaaron/serial-port) with the added bennefit of being a light-weight wrapper of PureJavaComm, instead of RXTX.  

Next up:  Firmata 
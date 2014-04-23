---
layout: post
title:  "clj-firmata 1.1.0 Released"
date:   2014-04-16 15:00:00
tags: software clojure arduino
comments: true
---

Following very hot on the footsteps of 1.0.0 (which I skipped announcing due to failingto properly sign it on Clojars), [clj-firmata 1.1.0](https://clojars.org/clj-firmata) is now available. This is a backward compatible release.

### New Features

* added `reset-board!` to send the reset command to the board
* added `version` and `firmware`
* added `firmata.shift` for operating with shift registers
* added `arduino-map`, `arduino-constrain` and `to-voltage` to `firmata.utils`

### Enhancements

* On opening the board, the version and firmware are read off the serial port and saved to the board state.  This allows the board to "settle" and then all messages sent to the board are properly accepted. 

### The 1.0.0 Release included: 

* complete Firmata protocol
* updated dependencies
* thread-safe use of the serial connection to the board
* `firmata.receiver` namespace for convenient event handling
* better use of `core.async` for routing events
* replaced `defrecord` uses with protocols and `reify`

The source can be found [here](https://github.com/peterschwarz/clj-firmata) and [samples](https://github.com/peterschwarz/clj-firmata-samples). 

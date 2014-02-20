---
layout: post
title:  "clj-firmata 0.1.0 Released"
date:   2014-02-19 21:00:00
tags: software clojure arduino
comments: true
---

With a little over a week between this and my [last release]({% post_url 2014-02-12-clj-serial-release %}), announcing the 0.1.0 release of [clj-firmata](https://github.com/peterschwarz/clj-firmata).  This handy little library let's one communicate with [Standard Firmata](http://firmata.org) devices (well, tested on an Arduino Uno)

The project is built on top of [clj-serial](https://github.com/peterschwarz/clj-serial) and [core.async](https://github.com/clojure/core.async).  The key idea here being that data is written to the serial port, and messages from the board are read back and placed onto an async channel.  The benefit of using a channel keeps the state management of the board at a minimum, allows the user to utilize core.async's filtering features. 

### So we can write data...

A simple example in the REPL:

``` clojure
    user=> (use 'firmata.core :reload)
    nil
    user=> (def board (open-board "tty.usbmodemfd131"))
    #'user/board
    user=> (set-digital board 13 :high)
    nil
    user=> (close board)
    nil
```

This will connect to a board at `tty.usbmodemfd131`, turn on the built in LED (at pin 13), and disconnect from the board.  
    
From here, we can already do more interesting things:

``` clojure 
(defn blink13 [board] 
  (set-digital board 13 :high)
  (Thread/sleep 500)
  (set-digital board 13 :low)
  (Thread/sleep 500))
  
(def board (open-board "tty.usbmodemfd131"))

(while true (blink13 board))
```

This will blink the LED at pin 13 ad nauseum. 

### But what about reading? 

Reading is pretty easy as well.  The board has an async channel, which may be used to pull events as they arrive. 

We can setup an event receiver like so: 

``` clojure 
(require '[clojure.core.async :as a])

(defn create-event-receiver
  [board event-handler]
  (a/go (loop [event (<! (:channel board))]
          (when event 
            (event-handler event)
            (recur (<! (:channel board)))))))
            
(def board (open-board "tty.usbmodemfd131"))

(def receiver (create-event-receiver board #(println %)))

```  

The `create-event-receiver` function is included in the `firmata.util` namespace. Right away, we'll see the following events:

``` clojure
{:type :protocol-version, :version 2.3}
{:type :firmware-report, :version 2.3, :name StandardFirmata.ino}
```
both of which are (it appears - this isn't documented) part of the firmata handshake - these may be consumed before returning the board in future releases. 

If we want to receive some analog data (say from a potentiometer attached to `A0`), we can do the following:

```clojure
 (enable-analog-in-reporting board 0 true)
 ```
  
and with our above `println` handler, we'll see a deluge of the following events:

```clojure
{:type :analog-msg, :pin 0, :value 1023}
```    

Presumably, you'll want to filter this messages and do something more sophisticated with the events as they arrive.

### On buildng the library

One thing that really help move this project along so quickly was building building a clean separation between the Firmata messages and the serial library.  This allowed for some very thorough unit testing (98% test coverage), with the REPL used to verify assumptions about the bit-twiddling.   It also provides reasonable documentation on how to use the library (for the most part).

Most of the dfficulty was in the protocol specs lack of examples for the messages, coupled with the inconsistent use of ports and pins between various messages (I'm looking at you `DIGITAL_IO_MESSAGE` and `PIN_STATE_REPLY`).   I ended up looking at a number of libraries to figure out how it all works, though predominately getting help from [this java class](http://playground.arduino.cc/uploads/Interfacing/Arduino.java.txt) posted at Arduino, and [tfirmata](https://github.com/pdt/tfirmata) (which is surprising, since I haven't written an byte of TCL).

All in all, it really hammered home the idea of how rapidly productive one can be writing libraries in Clojure. 
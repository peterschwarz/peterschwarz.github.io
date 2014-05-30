---
layout: post
title:  "Clojure REPL-Friendly Raspberry PI GPIO"
date:   2014-05-30 15:30:00
tags: software clojure RaspberryPI 
comments: true
---

Following on to my last post about [GPIO on the Raspberry PI]({% post_url 2014-04-23-blink-a-hello-world %}), I continued my exploration to reading and listening to value changes on a GPIO pin. 

I found that the [PI4J](http://pi4j.com/) library was not REPL friendly.  I could close a port and shutdown my use of the `GpioFactory`, but my listeners wouldn't be cleaned up. Once I created a listener once, that listener was stuck (and orphaned).  Looking at the changes in PI4J's [git repo](https://github.com/Pi4J/pi4j/), this looks fixed on a development branch, but far from release. 

I decided to circle back and start from scratch.  Here's where [clj-gpio](http://github.com/peterschwarz/clj-gpio) was born. 

Writing to the ports was relatively easy - simply writing the correct to the various `/sys/class/gpio` files.  Reading the current value was easy, too (yet another file read).  

Things get a little more hairy when you need to be notified that the file has changed. One can poll for changes, but this isn't very timely, nor is it very efficient. 

So what we need is an OS-level mechanism for watching for file changes. Enter [`epoll`](https://en.wikipedia.org/wiki/Epoll). This is a linux system call, so it's understandable that we don't have access to it through the JDK. 

Using [JNA](https://github.com/twall/jna), we can access these system calls [directly](https://github.com/peterschwarz/clj-gpio/tree/master/src/main/java/io/bicycle/epoll) (of course only on systems where they are available) with little effort - no JNI needed. Also, thanks to Clojure's Java interopt, using this small set of classes is relatively [straight-forward](https://github.com/peterschwarz/clj-gpio/blob/master/src/main/clojure/gpio/core.clj#L130).  Coupling this with core.async, and now we have a very simple gpio pin listening tool.

Now, in our REPL, we can do the following: 

    user=> (def ch-port (open-channel-port 18))
    #'user/ch-port
    user=> (set-direction! ch-port :in)
    nil
    user=> (set-edge! ch-port :both) ; or :falling, :rising, and :none to disable 
    nil
     user=> (def ch (create-edge-channel ch-port))
    #'user/ch
    user=>  (require '[clojure.core.async :as a :refer [go <!]])
    nil
    user=> (go (loop [] (when-let [value (<! ch)] (println value) (recur))))
    #<ManyToManyChannel clojure.core.async.impl.channels.ManyToManyChannel@1197ad0>
    
When we're done with our listener and our port we'll can just do

    user=> (release-edge-channel! ch-port ch)
    nil
    user=> (close! ch-port)
    nil
    
This can be done repeatably with in a single REPL session. 

Up next for this little library?  [PWM](https://en.wikipedia.org/wiki/Pulse-width_modulation)!
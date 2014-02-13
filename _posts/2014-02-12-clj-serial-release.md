---
layout: post
title:  "clj-serial 1.0.0 Released"
date:   2014-02-12 18:00:00
tags: software clojure arduino
comments: true
---

Announcing the 1.0.0 release of [clj-serial](https://github.com/peterschwarz/clj-serial).  As mentioned, it's forked from [serial-port](https://github.com/samaaron/serial-port) with the following changes:

* Updated to Clojure 1.5.1
* Backed by [PureJavaComm](https://github.com/nyholku/purejavacomm)
* removed the various `write-` functions in favor of one that takes arbitrary values of number, byte arrays or sequences of numbers 

Possible future changes:

* Remove the index-based port selection - bit odd, and really only works well in a REPL
* Consolidate and simplify the various `listen`-related functions 

Deploying to clojars and promoting the jar was remarkable easy, thanks to a post by [Michael Peterson](http://thornydev.blogspot.com/2013/03/signing-and-promoting-your-clojure.html)
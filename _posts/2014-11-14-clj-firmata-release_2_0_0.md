---
layout: post
title:  "clj-firmata 2.0.0 Released"
date:   2014-11-14 15:54:00
tags: software clojure arduino
comments: true
---

Announcing the 2.0.0 Release of [clj-firmata](https://github.com/peterschwarz/clj-firmata/releases/tag/clj-firmata-2.0.0).  

This is a huge release in that it both supports connecting to Firmata devices over arbitrary TCP sockets, it also is now available in ClojureScript on Node.js. 

###Major Changes

* Update Libraries to Clojure 1.6 (Issue #9)
* ClojureScript Support on Node.js (Issue #4)
* Connect to Firmata devices over TCP/IP (Issue #1)

###Minor Changes

* Allow for Firmata protocol extension (Issue #6)
* Configurable High/Low return values (Issue #7)

###Breaking changes

* Removed `firmata.shift`  (Moved to [partsbox](https://github.com/peterschwarz/partsbox))
* Some API name changes (e.g. `open-board` -> `open-serial-board`) 

Thanks to [blakejakopovic](https://github.com/blakejakopovic) for design input and testing help. 
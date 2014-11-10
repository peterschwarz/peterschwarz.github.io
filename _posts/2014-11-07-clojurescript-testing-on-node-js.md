---
layout: post
title:  "ClojureScript Testing on Node.js"
date:   2014-11-07 14:23:00
tags: software clojure  
comments: true
---

I've been working on and off adding ClojureScript support to [clj-firmata](https://github.com/peterschwarz/clj-firmata) on [Node.js](http://nodejs.org).  Through the use of [cljx](https://github.com/lynaghk/cljx), and the fact that [core.async](https://github.com/clojure/core.async) supports both environments, the conversiont process was actually pretty smooth.  All save one particular area: unit tests. 

There is a great [port](https://github.com/cemerick/clojurescript.test) of clojure test, which does that job very well (though, converting tests that were synchronous-blocking to asyncronous is quite a bit harder). Where I struggled was finding the correct settings to get things to run at all.

The following lein project configuration works for tests:

```
...
cljsbuild {:builds [{:id "test"
                     :source-paths ["src/cljs" "test/cljs" "target/classes" "target/test-classes"]
                     :compiler {:output-to   "target/testable.js"
                                :output-dir  "target/test-js"
                                :target :nodejs
                                :optimizations :simple
                                :hashbang false}}]
           :test-commands {"unit-tests" ["node" :node-runner
                                         "target/testable.js"]}})
...
```

The kicker, in the end, was the `:hashbang false` setting.  This would cause the test run to fail with the message `ERROR: cemerick.cljs.test was not required.`.  

Also, debuging is rather horrific. Source-map support does not work when testing (it results in the same message).

Still, as a TDD developer, it's quite important to see the green bar.
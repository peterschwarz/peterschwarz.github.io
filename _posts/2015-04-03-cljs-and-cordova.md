---
layout: post
title:  "ClojureScript, Om and Cordova: Getting Started"
date:   2015-04-03 13:55:00
tags: software clojurescript om cordova mobile
comments: true
---

Running Clojure on mobile sounds like quite a pipe dream. If you were to ask the folks at [Android ♥ Clojure](http://clojure-android.info/), you'd be told it's a "hobby project" (read: not ready for prime-time).  On iOS?  Don't even think about it. 

Fortunately, we have the ability to run JavaScript on mobile, thanks to tools like [Cordova](https://cordova.apache.org/).  Where there's JavaScript, there's ClojureScript. Great right?

It _is_ absolutely great, but getting started has a few hoops to jump through.  Even more hoops are needed to get the an effective development workflow, but I'll go over that in a later post. 


### Setting up your ClojureScript/Om project

This is easy.  I recommend just using a basic Leinigen template - [mies-om](https://github.com/swannodette/mies-om) does the trick.

```
> lein new mies-om hello-world
```

Don't worry too much about where you put the code, as we're going to have copy some items over to another project soon enough. I'll refer to this directory as `hello-world` from here out.

### Setting up your Cordova project

I won't walk you through installing Cordova it's very straight forward.  I'll just say that much of the following has all the node dependencies installed locally (I prefer having all my projects as self-contained as possible).

Creating a new project is very easy. 

```
> cordova create cordova-hello-world com.example.hello HelloWorld
```

Note, having a package name, with a classic format (e.g. `com.example.whatever`) here is important if you're using the Android platform.  It will not build the Android artifacts without one.

There you go, now you have two projects, on in `hello-world` and one in `cordova-hello-world`.

Wait, two projects?  That doesn't seem quite right.  We want one project one with our ClojureScript code and our Cordova configuration. 

### Mix and match

First things first, we'll want to bring our ClojureScript code into our Cordova project.  This is as easy as coping the `src` folder from `hello-world` to `cordova-hello-world`.  We'll also copy the `project.clj` into `cordova-hello-world`. 

Now, we need to adjust the ClojureScript compilation output.  We'll modify our `project.clj` like so:

``` clojure

:clean-targets ["www/js/app"]

:cljsbuild {
  :builds [{:id "starter"
                :source-paths ["src"]
                :compiler {
                  :output-to "www/js/app/hello.js"
                  :output-dir "www/js/app"
                  :optimizations :advanced
                  ; :externs ["src/hello/externs.js"] ; if you have any, or use plugins
                  :cache-analysis true
                  :source-map "www/js/app/hello.js.map"}}]}

```

We'll use advanced compilation in this case, as the application will much more quickly on devices. 

Now we'll need change our base HTML in `www/index.hml` to use the clojurescript output: 

``` html
    <body>
        <div id="app" class="app"/>
        <script type="text/javascript" src="cordova.js"></script>
        <script src="js/app/hello.js" type="text/javascript"></script>
    </body>
```

### Building and Running

We're ready to build our app, but not quite ready to run. We'll add the [browser platform](https://www.npmjs.com/package/cordova-browser) for our development workflow. 

```
> cordova platform add browser
```

Now to build and run the app, run the following couple of commands in our `cordova-hello-world` directory:

```
> lein cljsbuild once
Compiling ClojureScript.
Compiling "www/js/app/hello.js" from ["src"]...
Successfully compiled "www/js/app/hello.js" in 28.02 seconds.

> cordova run browser
```

This will open up a browser window with your app, and its lovely "Hello World" message.  We're all set to start building out functionality into our fresh, new Om mobile app.

Next up, making our workflow actually bearable and productive.  




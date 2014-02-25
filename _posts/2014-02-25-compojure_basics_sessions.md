---
layout: post
title:  "Compojure Basics: Sessions"
date:   2014-02-25 11:00:00
tags: software clojure
comments: true
---

[Compojure](https://github.com/weavejester/compojure) is a great routing library for [Ring](https://github.com/ring-clojure/ring) for building web applications.  The documentation is pretty good too - documentation being a pain point for me with many Clojure libraries.  One area I've had a bit of trouble understanding was how to access and update HTTP session values. 

Let's take a look at a very simple compojure handler:

{% gist 9211799#file-handler-clj %}

Let's break this down from the top level:

We've started by wrapping the handlers with all sorts of Ring middleware with the use of `(handler/site app-routes)`.  [Under the covers](https://github.com/weavejester/compojure/blob/master/src/compojure/handler.clj#L28-L46), this calls `wrap-session` (among other things), which adds the session to the map of request-related information ultimately provided to our handlers.

```clojure 
(defroutes app-routes
  (GET "/name/" {session :session} (handler session "World"))
  (GET "/name/:name" [name :as {session :session}] (handler session name))
 ...
```

We setup a couple of routes, both of which need a session.  We're getting a handle on the session in two different ways.  The first, in which we're defaulting the name to "World", we are using [Clojure destructuring](http://blog.jayfields.com/2010/07/clojure-destructuring.html) to get access to the session directly from the incoming request map. 

The second is using a slightly more sophisticated bit of destructuring.  The vector ` [name :as {session :session}]` combines a couple of things into one neat little form. The first part, `name` are the path parameters.  If we weren't worried about the session, we could just write this as `[name]`, making things look very much like a standard function. The second part, beginning with the `:as`, is simply our previous destructuring. Indeed, if we look at the [docs](https://github.com/weavejester/compojure/wiki/Destructuring-Syntax), we can do a wide variety of things with this destructuring syntax.

So now that we have our sesion, we can operate on it just like any other map.  In our case, we want to grab a visit count in our handler:

```clojure
(defn handler
  [session name]
  (let [count (:count session 0)]
     ...
```

In this one call, we are also able to default the visit count value on a brand new session.

If we left it at this, though, we'd be wondering why we're always getting zero for our visit count.  Well, this is clojure and the Session needs to be updated in a functional way. 

```clojure
	...
	{:session (assoc session :count (inc count))})))```

We're incrementing the value and calling `assoc` to create a new map with the new count value.  We then assign our new session map to the session on the response, which the Ring middleware will take and work it's magic (I really want to start writing "majic", thanks to all the 'j's in Clojure). 

Finally, way down in our content generator, we see: 

```clojure
(defn generate-response [data & [status]]
  {:status (or status 200)
   :body (str "<div>Hello <strong>" (:hello data) "</strong></div><div>We seen you <em>" (inc (:count data)) "</em> time(s)</div>")})
```

Which is simply creating the basic response scaffolding for our endpoint.  We give it a status, and some body content.  We put it in a map (which also explains our merge later) and ultimately return this up the chain of middleware to be supplied to the web client. 

Now we've worked our way throught the simple task of getting access to a session in a Compojure route.  Along the way, we've also figured out a bit more about Clojure's destructuring syntax.  Very powerful, yet very straightforward, isn't it?    
---
layout: post
title: Compojure on GAE - What's this about?
category: compojureongae
---

I'm interested in the Clojure programming language and the Google App
Engine platform right now, and I'm playing around  with the
combination of both.  Since there isn't a whole lot of information on
this on  the net, I thought I'd share the results of my experiments in
this blog. Here's the rundown on what this will (and won't) be about:

## Clojure

Clojure is a functional programming language of the Lisp family,
designed with concurrency in mind to make it easy to write
multi-threaded programs. It runs on the Java Virtual Machine and can
call (and be called from) Java code. I'm not going to write a tutorial
on programming Clojure (many others can do this better than me) nor am
I going to try to convince you that Clojure is the greatest language
in the world. I just think it's very interesting and I assume you do,
too - otherwise you probably wouldn't have stumbled across this
site. If you want to learn more about Clojure, the [official
website](http://www.clojure.org) 
provides very good documentation. I highly recommend watching the
[videos](http://clojure.blip.tv/) of Rich 
Hickey's talks, they got me hooked in the first place. 

## Compojure

Compojure is one of the frameworks for developing web applications in
Clojure that are being developed right now. I don't know if it's the
best, but it's the one I'm playing around with. The documentation
isn't very extensive, but you can find some at [http://preview.compojure.org] and there's an active [Google group](http://groups.google.com/group/compojure). You can grab the code at [GitHub](http://github.com/weavejester/compojure). For getting started I recommend this [short tutorial](http://groups.google.com/group/compojure/browse_thread/thread/3c507da23540da6e) by Compojure's creator, James Reeves.

## Google App Engine

[Google App Engine](http://appengine.google.com/) (GAE) is a service provided by Google that let's you develop web applications that can be deployed to Google's server infrastructure. Google provides several APIs that applications can use, e.g. for user authentication and storing data in a distributed data store. The idea is that applications can easily scale to handle everything from very small to very large loads. When App Engine was initially released it only provided a Python runtime environment. In April 2009 the Java runtime for App Engine was introduced. This opened the door not only for Java programs but for a number of languages that run on the JVM, including Clojure.

## The example app: A blog! (yawn)

I chose to implement a little blogging application to experiment with Clojure/Compojure on GAE. This isn't very exciting at all and I don't intend to actually use it. But it is simple and straightforward and displays much of the functionality of a typical web app: displaying pages, handling form input, storing and retrieving persistent data and authenticating users. (Edit: After reading [this question](http://stackoverflow.com/questions/471940/why-does-every-man-and-his-dog-want-to-code-a-blogging-engine) on stackoverflow I feel a little bad about my choice of example app. But at least I'm in good company...)

So, that's the plan. In the next post I'll talk about setting up the development environment.

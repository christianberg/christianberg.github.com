---
layout: post
title: Setting up the Dev Environment
category: compojureongae
---

In this post I'll cover the setup I use for developing my Compojure/GAE app.

<!--more-->

## Emacs

Of course you can use any editor you like to edit your source code,
but Emacs has some great features for editing Lisp code that make
developing in Clojure a pure joy. When properly set up, you can add
new or modify existing code and inspect all your data while the
application is running.

I followed the instructions at <http://technomancy.us/126> to set up
Slime/Swank for Clojure and it worked flawlessly without any
modifications. I can also recommend the [Emacs Starter Kit][1] that is
mentioned in the above post.

## Clojure

There are several options for getting the clojure.jar that you'll need:

- Download it from <http://code.google.com/p/clojure/downloads/list>
- Clone the git repository at <http://github.com/richhickey/clojure> 
  and build from source using ant
- If you're using Emacs and followed the instructions above, you can 
  automatically download the source and build it using the
  `clojure-install` command.
- Compojure provides a zip file of its dependendies (see below), 
  this also includes Clojure.

## Compojure

You can download a tarball of the Compojure sources from
<http://github.com/weavejester/compojure/downloads>, but I'd recommend
cloning the git repository at
<http://github.com/weavejester/compojure>. Either way you'll build
the compojure.jar from source using ant. 

Compojure has a few dependencies. You can download a zip file
containing all needed jars (including clojure.jar and
clojure-contrib.jar) from the download link above, or you can just run
`ant dep`, which will download the zip for you.

## Google App Engine SDK

Download the latest GAE SDK from
<http://code.google.com/appengine/downloads.html>. Choose the SDK for
Java and unzip it somewhere.

In the next post I'll create a simplistic Hello World Compojure app.

[1]: http://github.com/technomancy/emacs-starter-kit

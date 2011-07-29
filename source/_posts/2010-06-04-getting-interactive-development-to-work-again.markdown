---
layout: post
title: Getting Interactive Development to Work (Again)
category: compojureongae
---

In my [last post][0] I promised to fix my local development setup to
enable the interactive development style typical for Clojure and make
the App Engine services (such as the datastore) available from the
REPL.

Others have already tackled the same problem. The best resource I
found is the [hackers with attitude blog][1], another one is
[here][2].  My code is largely based on these contributions, I rolled
my own version mainly to get a better understanding of the setup.

<!--more-->

## Initializing the App Engine services

To use the App Engine APIs outside of the Google servers, a local
ApiProxy and Environment needs to be provided. While the ApiProxy
needs to be initialized only once per JVM, the Environment needs to be
set for every thread on which API calls are made. This is trivial for
the REPL, which runs in one thread. It's a bit more work when you want
to run a local Jetty server, since it spawns new threads for handling
requests. Fortunately, the design of Ring makes it easy to add
so-called middleware to an existing web app that can handle the
environment setup.

Enough said, here's the code. I'll go through it def by def below.

{% gist 425238 %}

The code is in a separate namespace, so it doesn't get AOT-compiled
and deployed with the rest of the app. I'm using an atom to store the
Jetty server instance. Using `defonce` was helpful while developing
this, because I could recompile the file (`C-c C-k` in Emacs) without
losing the reference to the running Jetty server.

The two functions `set-app-engine-environment` and
`set-app-engine-delegate` do the necessary setup work on the
per-thread and per-jvm basis, respectively.

`init-app-engine` just calls these two functions. It's intended to be
called from the REPL, after which you're able to use API calls like
`create-entity` in the REPL.

`wrap-local-app-engine` is a Ring middleware that sets the environment
for the current thread before passing the request on to the wrapped
Ring (or Compojure) app.

The `start-server` function takes a Ring app, does the per-JVM setup,
wraps the app with the middleware for the per-thread setup and starts
a Jetty server running the app. If there already is a Jetty server
stored in the atom, it is stopped first, so you can use the function
to restart the Jetty as well. The `:join? false` argument is
important, otherwise the call to `run-jetty` will not return.

I also added the `wrap-file` middleware to serve static files (and
`wrap-file-info` to add Content-Type and Content-Length headers). This
mimics the behaviour of the Google servers, which by default serve all
files included in the war directory. (Note that, unlike the Google
servers, this setup also serves the files in the WEB-INF directory. In
a production system that would be a security concern, for local
development I don't mind.)

Last (and also least interesting), the `stop-server` function stops
the Jetty server (and sets the atom back to nil).

## Setting up the classpath

To use the local API implementations we need some additional jars on
the classpath. Here's the updated project.clj:

{% gist 425239 %}

The new dependencies go into `dev-dependencies`, since they mustn't be
deployed with the app. However, in the current development version of
Leiningen 1.2 (which separates dependencies and dev-dependencies into
different directories), the dev-dependencies are apparently only
intended for Leiningen plugins such as swank-clojure - they are not
put on the classpath for the REPL. I had to patch Leiningen to make
this work. Here's the diff:

{% gist 425246 %}

I'll try to get this (or something similar) into Leiningen. **Update:
I wasn't the only one with this problem. It was recently patched in
Leiningen.** Stable Leiningen (1.1) will probably work out of the box
- I haven't tried.

Running `lein deps` installs the new dependencies. As I said in my
last post, you'll likely have to manually install the jars from the
App Engine SDK into your local Maven repository. Here's an example
command for one of the jars:

{% gist 425284 %}

## Putting it to use

Run `lein swank`, enter `M-x slime-connect` in Emacs to connect to the
REPL and code away as usual. To call functions that make use of the
App Engine API, enter this in the REPL:

{% highlight clojure %}
(require 'local-dev)
(local-dev/init-app-engine)
{% endhighlight %}

To start a Jetty server, just enter:

{% highlight clojure %}
(local-dev/start-server (var example))
{% endhighlight %}

`example` is the name of the Compojure app defined by `defroutes` in
core.clj.  **Update: Using `var` here allows you to change the
definition of `example` itself without having to restart the server.**

## What's next?

The next thing I'm working on is using the Users API for
authentication and authorization (instead of the simple
security-constraint method described in my [last post][0]).  I'll need
to make some changes to this local-dev code in order to properly test
that locally. So stay tuned...

As always, questions and suggestions are very welcome in the comments
section below!

[0]: /blog/2010/06/01/accessing-the-app-engine-datastore
[1]: http://www.hackers-with-attitude.com/2010/04/clojure-google-app-engine-setup-update.html
[2]: http://carpathia.blogspot.com/2010/05/yet-another-clojure-compojure-google.html

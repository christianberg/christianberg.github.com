---
layout: post
title: Getting Interactive Development to Work (Again)
category: compojureongae
---

In my [last post](http://compojureongae.posterous.com/accessing-the-app-engine-datastore),
I promised to fix my local development setup to enable the interactive
development style typical for Clojure and make the App Engine services
(such as the datastore) available from the REPL.

Others have already tackled the same problem. The best resource I
found is the [hackers with attitude
blog](http://www.hackers-with-attitude.com/2010/04/clojure-google-app-engine-setup-update.html),
another one is [here](http://carpathia.blogspot.com/2010/05/yet-another-clojure-compojure-google.html).
My code is largely based on these contributions, I rolled my own
version mainly to get a better understanding of the setup.

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

{% highlight clojure %}
(ns local-dev
  "Tools for local development.
   Enables the use of the App Engine APIs on the REPL and in a local
   Jetty instance."
  (:use ring.adapter.jetty
        [ring.middleware file file-info])
  (:import [java.io File]
           [java.util HashMap]
           [com.google.apphosting.api ApiProxy ApiProxy$Environment]
           [com.google.appengine.tools.development
            ApiProxyLocalFactory
            LocalServerEnvironment]))

(defonce *server* (atom nil))
(def *port* 8181)

(defn- set-app-engine-environment []
  "Sets up the App Engine environment for the current thread."
  (let [att (HashMap. {"com.google.appengine.server_url_key"
                       (str "http://localhost:" *port*)})
        env-proxy (proxy [ApiProxy$Environment] []
                    (isLoggedIn [] false)
                    (getRequestNamespace [] "")
                    (getDefaultNamespace [] "")
                    (getAttributes [] att)
                    (getAppId [] "_local_"))]
    (ApiProxy/setEnvironmentForCurrentThread env-proxy)))

(defn- set-app-engine-delegate [dir]
  "Initializes the App Engine services. Needs to be run (at least) per
  JVM."
  (let [local-env (proxy [LocalServerEnvironment] []
                    (getAppDir [] (File. dir))
                    (getAddress [] "localhost")
                    (getPort [] *port*)
                    (waitForServerToStart [] nil))
        api-proxy (.create (ApiProxyLocalFactory.)
                           local-env)]
    (ApiProxy/setDelegate api-proxy)))

(defn init-app-engine
  "Initializes the App Engine services and sets up the environment. To
  be called from the REPL."
  ([] (init-app-engine "/tmp"))
  ([dir]
     (set-app-engine-delegate dir)
     (set-app-engine-environment)))

(defn wrap-local-app-engine [app]
  "Wraps a ring app to enable the use of App Engine Services."
  (fn [req]
    (set-app-engine-environment)
    (app req)))

(defn start-server [app]
  "Initializes the App Engine services and (re-)starts a Jetty server
   running the supplied ring app, wrapping it to enable App Engine API
  use
   and serving of static files."
  (set-app-engine-delegate "/tmp")
  (swap! *server* (fn [instance]
                   (when instance
                     (.stop instance))
                   (let [app (-> app
                                 (wrap-local-app-engine)
                                 (wrap-file "./war")
                                 (wrap-file-info))]
                     (run-jetty app {:port *port*
                                     :join? false})))))

(defn stop-server []
  "Stops the local Jetty server."
  (swap! *server* #(when % (.stop %))))
{% endhighlight %}

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

{% highlight clojure %}
(defproject compojureongae "0.2.0"
  :description "Example app for deployoing Compojure on Google App
  Engine"
  :namespaces [compojureongae.core]
  :dependencies [[compojure "0.4.0-RC3"]
                 [ring/ring-servlet "0.2.1"]
                 [hiccup "0.2.4"]
                 [appengine "0.2"]
                 [com.google.appengine/appengine-api-1.0-sdk "1.3.4"]
                 [com.google.appengine/appengine-api-labs "1.3.4"]]
  :dev-dependencies [[swank-clojure "1.2.0"]
                     [ring/ring-jetty-adapter "0.2.0"]
                     [com.google.appengine/appengine-local-runtime "1.3.4"]
                     [com.google.appengine/appengine-api-stubs "1.3.4"]]
  :compile-path "war/WEB-INF/classes"
  :library-path "war/WEB-INF/lib")
{% endhighlight %}

The new dependencies go into dev-dependencies, since they mustn't be
deployed with the app. However, in the current development version of
Leiningen 1.2 (which separates dependencies and dev-dependencies into
different directories), the dev-dependencies are apparently only
intended for Leiningen plugins such as swank-clojure - they are not
put on the classpath for the REPL. I had to patch Leiningen to make
this work. Here's the diff:

{% highlight diff %}
diff --git a/src/leiningen/classpath.clj b/src/leiningen/classpath.clj
index 3be7e1f..836740e 100644
--- a/src/leiningen/classpath.clj
+++ b/src/leiningen/classpath.clj
@@ -8,7 +8,9 @@
   "Returns a seq of Files for all the jars in the project's library directory."
   [project]
   (filter #(.endsWith (.getName %) ".jar")
-          (file-seq (file (:library-path project)))))
+          (concat
+           (file-seq (file (:library-path project)))
+           (file-seq (file (str (:root project) "/lib/dev"))))))
 
 (defn make-path
   "Constructs an ant Path object from Files and strings."
{% endhighlight %}

I'll try to get this (or something similar) into Leiningen. **Update: 
I wasn't the only one with this problem. It was recently patched in Leiningen.** 
Stable Leiningen (1.1) will probably work out of the box - I haven't tried.

Running `lein deps` installs the new dependencies. As I said in my
last post, you'll likely have to manually install the jars from the
App Engine SDK into your local Maven repository. Here's an example
command for one of the jars:

{% highlight sh %}
mvn install:install-file -DgroupId=com.google.appengine \
-DartifactId=appengine-api-labs -Dversion=1.3.4 -Dpackaging=jar \
-Dfile=$GAESDK/lib/user/appengine-api-labs-1.3.4.jar
{% endhighlight %}

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

`example` is the name of the Compojure app defined by `defroutes` in core.clj.
**Update: Using `var` here allows you to change the definition of `example` 
itself without having to restart the server.**

## What's next?

The next thing I'm working on is using the Users API for
authentication and authorization (instead of the simple
security-constraint method described in my [last
post](http://compojureongae.posterous.com/accessing-the-app-engine-datastore)).
I'll need to make some changes to this local-dev code in order to
properly test that locally. So stay tuned...

As always, questions and suggestions are very welcome in the comments
section below!

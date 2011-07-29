---
layout: post
title: Deploying to App Engine
category: compojureongae
---

In my [last post][0], I set up a basic Hello World Compojure app,
running on a local Jetty instance. Now I want to deploy this to Google
App Engine.

<!--more-->

## Creating a Servlet ##

App Engine expects a standard Java web application, which means we
have to take a small step out of pure Clojure-land into the Java realm
and implement the HttpServlet interface. The defservice macro from the
ring API makes this trivial.

Obviously, Google runs their own app servers, so we don't need to
start a Jetty instance. The updated core.clj looks like this: 

{% gist 397298 %}

The project.clj file needs to be updated to reflect the changed
dependencies:

{% gist 397390 %}

Note that I added a `:namespaces` entry. This triggers the AOT
compilation of the Clojure source into Java bytecode. **[Edit]** I
also customized some paths - more on that below. **[/Edit]**

Google App Engine requires two config files, `web.xml` and
`appengine-web.xml`. For my simple app, these are pretty
straight-forward. The `web.xml` defines the mapping from URL patterns
to servlet classes. Here it is: 

{% gist 397401 %}

The `servlet-name` (`blog`) is a generic identifier, you can use
anything you like. The `servlet-class` needs to be the clojure namespace
that implements the `HttpServlet` interface - in this case
`compojureongae.core`. 

In the `appengine-web.xml` we set the GAE application id and an
arbitrary version string:

{% gist 397411 %}

I set up a github repository for my experiments at
<http://github.com/christianberg/compojureongae>.
The code as of the time of this blog post can be found at
<http://github.com/christianberg/compojureongae/tree/v0.1.1>.

## Building the war ##

**[Update]**
Sometimes things are much easier than they first appear. My initial
"build process" (using [leiningen-war][1]) was way too
complicated. Thanks to
<http://buntin.org/2010/03/02/leiningen-clojure-google-app-engine-interesting/>
for putting me on the right track. Here's how I do it now:

The deployment artifact for GAE is a standard java war file - actually
a war directory, i.e. an unzipped war file. This makes the build
process pretty trivial, you just have to adhere to the standard war
directory structure. This is accomplished by customizing the
`:library-path` and `:compile-path` in the `project.clj` (see
above). Building the project is simply done with the standard lein
commands:

{% codeblock %}
lein clean
lein deps
lein compile
{% endcodeblock %}

The current stable version of leiningen (1.1.0) mixes the dependencies
and the dev-dependencies. If you don't want the dev-dependency jars
included in your deployment, run this sequence of commands before
deploying:

{% codeblock %}
lein clean
lein deps skip
lein compile
{% endcodeblock %}

The development version of leiningen (1.2.0-SNAPSHOT) separates the
dev-dependencies into lib/dev, so you might want to check it out.
**[/Update]**

Now we have a war directory that can be used by the scripts that come
with the App Engine SDK. If you haven't yet, [download][2] it now.

## Testing the war ##

To make sure our war file is ok, let's test it locally. I unpacked the
SDK in the directory `$GAESDK`. Here's how to start the local server:

{% codeblock %}
$GAESDK/bin/dev_appserver.sh war
{% endcodeblock %}

You should see the familiar page at <http://localhost:8080/>

## Into the Cloud! ##

It's time to deploy. Just run

{% codeblock %}
$GAESDK/bin/appcfg.sh update war
{% endcodeblock %}

Enter your Google login when prompted and wait for the app to
deploy. (Remember that you need to create an Application in the GAE
admin dashboard first and put it's app id in the `appengine-web.xml`.)

The app is now live in the cloud. You can see my deployed version here:
<http://v0-1.latest.compojureongae.appspot.com/>

Have fun! If this inspires you to do your own experiments with Clojure
on App Engine, leave a comment below!

[0]: /blog/2010/05/07/a-fresh-start
[1]: http://github.com/alienscience/leiningen-war
[2]: http://code.google.com/appengine/downloads.html#Google_App_Engine_SDK_for_Java

---
layout: post
title: A Fresh Start
category: compojureongae
---

Since my last post a lot has happened in Clojure- and
Compojure-Land. Managing dependencies and building projects is much
easier now that there is [Leiningen][1] and more people
have played around with Clojure on Google App Engine, some are even
deploying live apps (check out [TheDeadline][2]). 

So I decided to build upon these great contributions and (finally)
continue this little tutorial.

<!--more-->

## Create a new project

From my last post, you can still read the part about Emacs, but you
can forget about getting all the dependencies. We'll use Leiningen for
that. To install Leiningen, follow the [installation instructions][1]. 
Then create a new project:

{% codeblock %}
lein new compojureongae
{% endcodeblock %}

This creates the basic directory structure with some skeleton
code. Edit the project.clj file to look like this:

{% gist 393313 %}

I removed the direct dependencies on clojure and clojure-contrib,
since depending on compojure automatically pulls these, but you could
leave them in (e.g. if you need a specific version). The
dev-dependency on lein-swank gives me integration with Emacs while
letting Leiningen handle the classpath config.

Running `lein deps` (in the directory containing project.clj)
downloads all required libraries and puts them in the lib directory.

## Start Hacking

Run `lein swank` to start a REPL, open Emacs and enter 
`M-x slime-connect` to connect to the REPL. Now we can start hacking
away in Emacs! Open `src/compojureongae/core.clj` and enter the
following: 

{% gist 393319 %}

(This is taken directly from Compojure's [Getting Started][3] page.)
Pressing `C-c C-k` compiles the file and starts the server - you can
see the output in the shell where you ran lein swank. Now browse to
<http://localhost:8080/> to see your first Compojure app.

Next step: Deploying this to Google App Engine!

[1]: http://github.com/technomancy/leiningen
[2]: http://the-deadline.appspot.com/
[3]: http://weavejester.github.com/compojure/docs/getting-started.html

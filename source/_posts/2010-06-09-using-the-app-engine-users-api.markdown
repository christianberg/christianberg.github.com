---
layout: post
title: Using the App Engine Users API from Clojure
category: compojureongae
---

In my previous post about [Accessing the Datastore][0] I set up basic
security using a `security-constraint` element in the deployment
descriptor (`web.xml`). This was simple, as the app didn't have to be
aware of security concerns at all. The downside is that the app
doesn't know if the user is logged in and can't react to that. For
example, the "Create new post" link is shown to all users, only after
clicking it (and logging in) they get an ugly error message about
missing privileges. This is bad usability, so let's use the App Engine
Users API and move the authentication and authorization into the app.

<!--more-->

## Setting up the routes

To make things easier, I changed my route definitions, separating the
public routes from those that need admin privileges. All admin route
URLs now start with `/admin`:

{% gist 431270 %}

The admin-routes are only allowed to be accessed by logged-in users
with admin privileges.  [appengine-clj][1] already comes with two
middleware functions that help with this: `wrap-with-user-info` adds
references to the UserService and (if a user is logged in) User
objects from the App Engine API to each
request. `wrap-requiring-login` checks that the user is logged in
before passing the request on to the wrapped handler - if not, the
user is redirected to the login page.

There's no `wrap-requiring-admin` (yet), so I quickly wrote it myself:

{% gist 431336 %}

`wrap-requiring-admin` depends on `wrap-requiring-login`, which in
turn depends on `wrap-with-user-info`, so I have to decorate my
`admin-routes` handler with all three:

{% gist 431338 %}

Finally, the routes are combined into my main handler:

{% gist 431343 %}

Why can't I just put `admin-routes` in there, just like
`public-routes`? The problem is, that the middleware I wrapped around
`admin-routes` jumps in before the route-matching. So even if
`admin-routes` can't match the request URL and passes on control to
the next handler, it first makes sure that the user is logged in as an
admin. In this case, the `not-found` handler (which always has to be
last) could only be reached by admins, all other users would have to
login and then get a 403 error when they enter a non-existing URL.
Therefor, I have to make sure that the `admin-routes` handler is only
called for URLs starting with /admin.

## Checking the users login status

So far, the new code does the same thing the old configuration did, I
haven't won anything. So let's make the site a little more dynamic and
change the output depending on the users login status. I changed the
sidebar to display information about the current user and login/logout
links. Also, the "Create new post" link is only shown for logged-in
admin users:

{% gist 431458 %}

(Note that `side-bar` now is a function, since the content is dynamic.)

I've achieved my goals: I can login and logout and I only see the
links I'm allowed to click. I can run this code using the local
dev_server and I can deploy it to the Google servers (see my 
[previous post][2] on how to do this).

But in the interactive development environment I set up in my 
[last post][3], nothing works! I'm always logged out and the login
link is broken.  Let's fix that.

## Making logins work in interactive development

The local implementation of the App Engine Users API calls an instance
of `ApiProxy$Environment`, which I have to provide, to figure out if a
user is logged in. In my last post, I set up a very minimal proxy,
that always answers this question with "no". Here's the relevant
snippet:

{% gist 431611 %}

This needs to be smarter. I decided to store information about the
current user globally in an atom. Of course, this implies that the
server can only be used by one user at a time - for a production
system this would be an incredibly stupid implementation, for local
development I think it's ok. Other options would be to store the login
information in session variables or directly in a cookie. Storing it
globally has the advantage, though,  that I can easily view and modify
the current login state from the REPL, which eases debugging (plus
it's simple to implement!).

Here's the definition of the atom holding the login information,
prefilled with some reasonable default values:

{% gist 431638 %}

The updated Environment proxy just reads from the atom:

{% gist 431643 %}

I added two helper functions to easily modify the atom:

{% gist 431645 %}

Now I can login and logout by calling the functions from the REPL and
the pages served by my Jetty server immediately reflect this. But the
login and logout links are still broken. I need to define handlers for
these:

{% gist 431653 %}

The `login-form` function just builds an exact copy of the login page
provided by the Google dev_server.

Last but not least, I have to update the `start-server` function to
combine these handlers with my app (the change is in line 9):

{% gist 431661 %}

That's all - a functioning local implementation of the Users API
complete with working login page. I hope you enjoy it!

As always, the complete source code can be found on [Github][4], the
version as of this writing is [here][5].  You can see the deployed app
[here][6] (of course I'm the only admin user, so you might want to try
it locally to see the full functionality...). Questions and
suggestions are very welcome in the comments below!

[0]: /blog/2010/06/01/accessing-the-app-engine-datastore
[1]: http://github.com/r0man/appengine-clj
[2]: /blog/2010/05/11/deploying-to-app-engine
[3]: /blog/2010/06/04/getting-interactive-development-to-work-again
[4]: http://github.com/christianberg/compojureongae
[5]: http://github.com/christianberg/compojureongae/tree/v0.3.0
[6]: http://v0-3.latest.compojureongae.appspot.com/

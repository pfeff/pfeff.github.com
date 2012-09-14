---
layout: post
title: "Building with Leiningen"
description: ""
category: 
tags: [clojure, leiningen]
---
{% include JB/setup %}

Leiningen is the defacto standard build tool for Clojure applications.  Like
most things I describe on this site, I won't go into the details or history of
any specific tool if that information is [available
elsewhere](https://github.com/technomancy/leiningen).

You should install Leiningen from the link above and then come back when you're
done.

...

Now that you've got it installed, we'll create a Compojure web application
based on a template built into Leiningen.

    lein new compojure todo-compojure

You should now have a minimal Compojure webapp in the todo-compojure folder.  Launch it with

    lein ring server

and visit it at [http://localhost:3000](http://localhost:3000).  It should
actually open automatically.

Next up, we'll take a tour of the generated code.

---
layout: post
title: "Serializing a Clojure Object"
description: ""
category: 
tags: [clojure, clojurescript]
---
{% include JB/setup %}

I lied.  In the last post I said I show how to serialize a Clojure Object to
JSON.  We're not going to do that.  It turns out that there's a much better way
when you're using Clojure on the server and ClojureScript in the browser: you
convert the clojure data structure to a string and then you eval it on the
other end.

Here's the data structure we're going to serialize:

    {:done false, :task "Write a ClojureScript webapp"}

In the handler, we're going to add a new route to respond to GET /task.

    (GET "/task" [] (pr-str {:task "Write a ClojureScript webapp" :done false}))

That's it!  You just call pr-str, and the data structure get's converted to a
string that can be eval'd (more on that later).

As always, [this code is available on Github](https://github.com/pfeff/todo-compojure/blob/fed8f14489e58bafcbebebb8e889ff2fd712e0dc/src/todo_compojure/handler.clj#L10).

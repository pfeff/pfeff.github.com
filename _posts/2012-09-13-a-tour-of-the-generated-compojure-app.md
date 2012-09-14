---
layout: post
title: "A tour of the generated Compojure app"
description: ""
category: 
tags: [clojure, compojure]
---
{% include JB/setup %}

In the previous installment, we generated a skeleton app based on a built in
Compojure template.  You can [read all about compojure
here](https://github.com/weavejester/compojure), but I won't discuss that now.
We'll have plenty of time for that later.

If you run the tree command in the newly created todo-compojure folder, you
should see a directory structure like so:

    .
    ├── project.clj
    ├── README.md
    ├── src
    │   └── todo_compojure
    │       └── handler.clj
    └── test
        └── todo_compojure
            └── test
                └── handler.clj

The Project File
----------------

Here's the generated project file:

<script src="https://gist.github.com/3719173.js"> </script>

The Handler
-----------

<script src="https://gist.github.com/3719208.js?file=gistfile1.clj"> </script>

The Test Case
-------------

And finally, the test case.

<script src="https://gist.github.com/3719214.js?file=gistfile1.clj"> </script>

Download It!
------------

I've [uploaded all of the above to GitHub](https://github.com/pfeff/todo-compojure/tree/2012-sept-13-tour-generated-compojure),
but it should be identical to what you just created.

Like most project skeletons, there really isn't much to see, but it is
minimally functional.  In the next installment, we'll go about creating a
simple task object and serving it as JSON.


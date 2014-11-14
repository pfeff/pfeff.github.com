---
layout: post
title: Testing an Om component
---

This post is a quick tutorial on testing an Om component.  I assume you know
what Om and clojurescript are.  I extracted this piece from a larger work in
progress which will hopefully be available soon.

If you go the the GitHub project page for Om, you will find a small section on
testing Om components.  This is basically just a pointer to a project built on
Om, and the implied message is that you should read the source.  Well, I went
ahead and did that for you and this is what I found.

Om tests fall in the category of integration tests.  Your tests are dependent
on a third party API (Om) and a complex runtime environment (a browser).  This
means that there''s going to be a non-trivial amount of setup.  By the end of
this post, you should have:

1. Configured Leiningen to build/execute your tests
2. A headless browser environment (slimerjs)
3. Some utilities for creating DOM nodes in which to run the tests
4. Some examples on which to base further work.

We''re going to use the test functionality built into cljsbuild to run our
tests.  This means adding a new map to the build vector to define how the test
code is built:

     {:id "tests"
      :source-paths ["src-cljs" "test-cljs"]
      :compiler {:pretty-print true
                 :output-dir "resources/private/js"
                 :output-to "resources/private/js/unit-test.js"
                 :preamble ["react/react.js"]
                 :externs ["react/externs/react.js"]
                 :optimizations :whitespace } }

If you''re familiar with building clojurescript already, this should be pretty
familiar.  The test code and the app code are being compiled into a single
javascript file named unit-test.js.  Since Om relies on the react library,
we''re telling cljsbuild to include the react source in the generated
javascript file.

Next, we need to add the test configuration itself:

    :test-commands {"unit-tests" ["node_modules/slimerjs/bin/slimerjs" :runner
                                  "resources/private/js/unit-test.js"
                                  ]}

This configuration tells cljsbuild to run our tests using slimerjs as the
javascript runtime.  The second member of the vector (:runner) will be replaced
by the clojurescript.test plugin at runtime with the path to a javascript test
runner provided by clojurescript.test.  Finally, our code to be tested is the
third element in the vector.  It will be passed to the runner script for
execution.

To make leiningen replace the :runner keyword with the path to the provided
script, add clojurescript.test to to your plugins vector.

  :plugins [[lein-cljsbuild "1.0.4-SNAPSHOT"]
            [com.cemerick/clojurescript.test "0.3.1"]
            [lein-npm "0.4.0"]]

You''ll notice that we also have lein-npm included in our plugins
configuration.  This allows us to include node.js dependencies in our project.
This allows us to include slimerjs in our project with just the following
configuration.

  :node-dependencies [[slimerjs "0.9.2"]]

With all of this in place, we now have a self contained build capable of
testing our application.

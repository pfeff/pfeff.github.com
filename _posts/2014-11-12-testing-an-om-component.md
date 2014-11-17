---
layout: post
title: Testing an Om component
---

This post is a quick tutorial on testing an Om component.  I assume you know
what Om and clojurescript are.  I extracted this piece from a larger work in
progress which will hopefully be available soon.

If you go the GitHub project page for Om, you will find a small section on
testing Om components.  This is basically just a pointer to a project built on
Om, and the implied message is that you should read the source.  Well, I went
ahead and did that for you and this is what I found.

Om tests fall in the category of integration tests.  Your tests are dependent
on a third party API (Om) and a complex runtime environment (a browser).  This
means that there's going to be a non-trivial amount of setup.  By the end of
this post, you should have:

1. Configured Leiningen to build/execute your tests
2. A headless browser environment (slimerjs)
3. Some utilities for creating DOM nodes in which to run the tests
4. Some examples on which to base further work.

Basic Setup
-----------

We're going to use the test functionality built into cljsbuild to run our
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

If you're familiar with building clojurescript already, this should be pretty
familiar.  The test code and the app code are being compiled into a single
javascript file named unit-test.js.  Since Om relies on the react library,
we're telling cljsbuild to include the react source in the generated
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

You'll notice that we also have lein-npm included in our plugins
configuration.  This allows us to include node.js dependencies in our project.
This allows us to include slimerjs in our project with just the following
configuration.

    :node-dependencies [[slimerjs "0.9.2"]]

With all of this in place, we now have a self contained build capable of
testing our application.

Some Utilities for Generating DOM
---------------------------------

In order to test an Om component, we'll need a little bit of DOM in which to
mount that component.  Specifically, we're going create a uniquely named div
and append it to the document body. We'll use Prismatic's Dommy for this.

The container's id:

    (defn new-id 
      ([]
       (str "container-" (gensym)))
      ([id]
       (str "container-" id)))

The div:

    (defn new-node [id]
      (-> (dommy/create-element "div")
          (dommy/set-attr! "id" id)))

And append it to the body:

    (defn append-node [node]
      (dommy/append! (sel1 js/document :body) node))

Finally, a utility to wrap this all up:

    (defn container!
      ([]
       (container! (new-id)))
      ([id]
       (-> id
           new-node
           append-node)))

Here's a usage example:

    (deftest test-container
      (let [c (container! "container-1")]
        (is (sel1 :#container-1))))

This code instantiates a container providing an ID.  This is useful for
testing, but you'll typically want to use the no-arg version that generates a
unique ID.

The Test Case
-------------

Now, here's the test case that's going to drive the first bit of implementation:

    (deftest video-element
      (let [c (container!)]
        (om/root core/video {} {:target c})
        (is (sel1 :video))))

This creates the container as described above, instantiates an Om component
(which we'll write shortly) named video, and asserts that the video element
exists in the page.

The Implementation
------------------

This test will fail, because we haven't defined our video component.  This
component just renders an empty video element by implementing the IRender
protocol.

    (defn video [data owner]
      (reify
        om/IRender
        (render [this]
          (dom/video nil nil))))

This should be sufficient to make our tests pass, but it's not all that
interesting.  In the next post, I'll test drive an interactive component.


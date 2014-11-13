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



---
layout: post
title: Use Webpack to compile ELM.
---

A pure Elm project uses its own toolchain for compilation.
I intend to combine ELM with another compile-to-javascript language (probably Typescript).
The plan is to use Webpack to combine everthing into a single bundle.

There is an Elm loader for webpack.
The documentation even tells you how to configure it.
It's not clear how to add it to an existing project.

This is what I came up with.

1. Create a plain JavaScript file named index.js.  Set it as your entry.
2. In index.js, load your Elm app:

    {% highlight javascript %}
    var Elm = require( '../elm/Main' ); //Elm files in a sibling folder
    Elm.Main.embed( document.getElementById( 'main' ) );
    {% endhighlight %}

3. Add the HtmlWebpackPlugin to your plugins array:

    {% highlight javascript %}
    plugins: [
        new HtmlWebpackPlugin({
            template: './path/to/index.html',
            inject:   'body',
            filename: 'dist/index.html'
        })
    ]
    {% endhighlight %}

When you run Webpack, it'll inject the bundle (including index.js) into index.html.


---
layout: post
title: How to draw a Wardley Map in JavaScript using D3.
---

I just pushed some [sample code up to Github](https://github.com/pfeff/wardley-map) that describes how to draw a Wardley Map using D3.
I want to emphasize that this is sample code and not a library.
You'll need to copy, paste, and hack.

Let me get you started.

First off, the relevant code is in src/ts/app.ts and src/html/index.html.
Everything else is plumbing.

This code uses D3's force directed layout with a couple customizations.

1. Nodes that appear first in the node list are closer to the top.
The layout is tree-like

2. The 'maturity' property on each node corresponds to the maturity area on the map.
This draws the node in the appropriate direction (aka focus).

{% highlight javascript %}
const simulation = d3f.forceSimulation()
    .force("link", d3f.forceLink().id(R.prop('id')))
    .force("charge", d3f.forceManyBody())
    .force("center", d3f.forceCenter(width/2, height/2))
    ;

simulation
    .nodes(nodes)
    .on("tick", ticked)
    ;

simulation
    .force("link")
    .links(links)
    ;

simulation.force("link").distance(R.always(60));
{% endhighlight %}

I've set the `id` property on `forceLink` to a function that returns the `id` property of the node.
This allows me to use names rather than ordinals.
The `R` is the Ramda library.

Next, I use SVG to draw the nodes and links.

{% highlight javascript %}
const link = svg.append("g")
    .attr("class", "links")
    .selectAll(".link")
    .data(links)
    .enter().append('line')
    .attr('class', 'link')
    ;

const node = svg.selectAll('.nodeg')
    .data(nodes)
    .enter().append('g')
    .attr('class', 'nodeg')
    ;

node.append("text")
    .attr("dx", 12)
    .attr("dy", ".35em")
    .text(R.prop('id'))
    ;

node.append('circle')
    .attr('class', 'node')
    .attr('r', 5)
    ;
{% endhighlight %}

This is pretty vanilla D3 for nodes, lines, and text.

Finally, the `ticked` function is where the magic happens.
It drives the layout.

{% highlight javascript %}
function ticked() {
    const k = 10*simulation.alpha();

    link
        .each(function(d) {
            d.source.y -= k;
            d.target.y += k;
        })
        .attr("x1", R.path(['source', 'x']))
        .attr("y1", R.path(['source', 'y']))
        .attr("x2", R.path(['target', 'x']))
        .attr("y2", R.path(['target', 'y']))
        ;

    node.attr("transform", function(d) { return translate(d.x, d.y) });
    node.each(gravity(focus(width), simulation.alpha()));
}
{% endhighlight %}

The chain of commands on the `link` object moves source nodes up and target nodes down.
This gives us the tree-like behavior.
It then draws a line between them.
I use a translate directive on the node position it.
Finally, the `gravity` and `focus` functions move the node horizontally into the appropriate column.

{% highlight javascript %}
function gravity(focus, alpha) {
    if ((alpha > 1) || (alpha <= 0)) {
        throw new RangeError("Alpha shoulde in range (0,1]: " + alpha);
    }
    return function(d) {
        if(d.maturity < 1 && d.maturity > 4) {
            throw new RangeError(d.maturity);
        }
        const previous = d.x;
        const next = d.x + (focus(d.maturity) - d.x) * alpha;

        d.x = next;
    }
}
{% endhighlight %}

Gravity moves a node incrementally in the direction of a focus.
In this case, focus is a position in a column.

{% highlight javascript %}
const focus = R.curry((width, n) => {
    const nthMiddle = middleOf(width);
    const foci = [0, nthMiddle(2), nthMiddle(3), width];
    return foci[n - 1];
});
{% endhighlight %}

I chose not to put the focus on the middle of each column because it let to too much clumping of the visualization.
Instead, the outer columns have their focus on their outer edges.

For completeness, these are the helper functions rerenced above.

{% highlight javascript %}
const focus = R.curry((width, n) => {
const middleOf = R.curry((width, n) => {
    return n * width/4 - width/8;
});

function translate(x, y) {
    return "translate(" + x + "," + y + ")";
}
{% endhighlight %}



---
title: A Generic Handler For AWS SDK Operations in Elm
layout: post
---

The code in this post is a work in progress.

I'm working on an app that uses the AWS SDK for JavaScript to query DynamoDB directly from the browser.
Yes, there are ways to securely do this.

Since the app is written in Elm, I need a driver in native JavaScript to perform the operation.
The naive way to do this is to write one handler per operation I want to expose in the SDK.
These functions are easily 10 lines of code.
Obviously, this does not scale.
Obviously, that's what I did first.

What I really want is a generic handler.
Here's my first attempt (in TypeScript):

    {% highlight javascript %}
    function wire(elmApp: ElmApp, clients: Clients) {
         elmApp.ports.awsRequest.subscribe(function(event: any) {
             var operationName = event.operation;
             var clientName = event.client;
             var params = event.params;
             clients[clientName][operationName](params, function(err, data) {
                 console.log("awsResponse handler");
                 if (err) {
                     elmApp.ports.jsError.send(err);
                 } else {
                     elmApp.ports.awsResponse.send(data);
                 }
             });
         });
     }
    {% endhighlight %}

The `wire` function does all the work of subscribing to requests and sending responses back to Elm.
It expects an object with `operation`, `client`, and `params` fields.
It then dynamically invokes the SDK function.
Here's how that works.

This is what's in the clients object:

    {% highlight javascript %}
     function initClients() {
         return {
             'DynamoDB': new Aws.DynamoDB.DocumentClient()
         };
     };
    {% endhighlight %}

If the elm code passes in 'DynamoDB' as the value of the client parameter, this function will invoke `operationName` on an instance of `DynamoDB`.
The `params` argument is passed unmodified to the operation.
When the operation invokes its callback, I send that data unmodified back to the elm handler.
It's up to the Elm to parse and dispatch the response as it sees fit.
That's a topic for another post though.

Finally, I need an entry-point to set all this up:

    {% highlight javascript %}
    export function elm(credentials: Aws.Credentials, elmApp: any) {
        console.log("Aws.elm");
        setCredentials(credentials, elmApp);
        var clients = initClients();
        wire(elmApp, clients);
    }
    {% endhighlight %}

If you liked this post, your followers might also.
Please give it a share!


---
layout: post
title: Elm Interop - Using a JavaScript Library
---

Motivation: I'm writing an Elm app, but I want to use Auth0's JavaScript library.

Elm does support JavaScript interop.
It's even [documented](https://guide.elm-lang.org/interop/javascript.html)!
If you're an Elm beginner, like me, the docs will leave you scratching your head.
Here's a more thorough example to get you started.

Note: I'm using webpack to compile and combine Elm and TypeScript into a single bundle.
If you're doing something different, I have no idea if this'll work.

## Step 1: Link Elm to TypeScript in your index.js

1. Require your TS and Elm apps
2. Instantiate your Elm App using `embed`
3. Pass you're Elm app to your TypeScript app

    {% highlight javascript %}
    var appTS = require('../ts/app.ts');
    var Elm = require('../elm/Main');
    var app = Elm.Main.embed(document.getElementById('main'));
    appTS.default(app)
    {% endhighlight %}

That default function is my TypeScript module's default export which I'll describe next.

## Step 2: Define a function that wires the library to Elm

The important bit here is that my Elm app is available as the elmApp parameter.
Per the Elm documentation, I can now call `send` and `subscribe` on port functions.
In this case, when my Elm app calls the `login` function I call Auth0's `lock.show()`.
Further, when Auth0 returns the profile object, I can `send` that object to my Elm app.
See the Auth0 docs for info on how to use the Auth0Lock API.

    {% highlight javascript %}
    export default function app(elmApp) {
         elmApp.ports.login.subscribe(function(event) {
             lock.show();
         });
         lock.on("authenticated", function(authResult) {
             lock.getProfile(authResult.idToken, function(error, profile) {
                 if (error) {
                     console.log("Authentication error: " + error);
                     return;
                 }
                 localStorage.setItem('id_token', authResult.idToken);
                 console.log(profile);
                 elmApp.ports.profile.send(profile);
             });
         });
     }
    {% endhighlight %}


## Step 3: Define the port functions in your Elm code

This step is easy.
The ports are just declarations.
They don't have a body.
The only gotcha: Elm seems to require an argument on the Cmd port.
I'm not using the argument, so I just pass in an empty string when I call `login`.

    {% highlight elm %}
    port login : String -> Cmd a
    port profile : (Profile -> msg) -> Sub msg
    {% endhighlight %}

The other argument types will depend on your application.
Profile is a record type that corresponds to the JavaScript object passed to `getProfile`.

## Step 4: Use the library from Elm

How you use the library is up to you ;).
Elm has a couple requirements though.

### Invoke Cmd functions from update

You'll need to invoke your Cmd port functions from `update`.
In my case, when a user clicks the login button, I generate a `Login` message.
Then, in `update`, I handle that `Msg` in a case statement:

    {% highlight elm %}
    Login -> (model, login "")
    {% endhighlight %}

I don't update the model because login happens asyncronously.

### Subscribe to JavaScript functions

Since the JavaScript code will call `profile` asyncronously, I need to declare a handler funciton in subscriptions:

    {% highlight elm %}
     -- SUBSCRIPTIONS

     loginComplete : Profile -> Msg
     loginComplete profile =
         LoginComplete profile

     subscriptions : Model -> Sub Msg
     subscriptions model =
         Sub.batch
             [ profile loginComplete
             ]
    {% endhighlight %}

This basically says to pass `profile` messages to the `loginComplete` function.
The `loginComplete` function returns a `LoginComplete` message containing the profile record.
Since `LoginComplete` is a member of the `Msg` type, you can handle it in `update`.

### Register subscriptions in main

The `subscriptions` function must be passed to your main function along with `init`, `update`, and `view`.

    {% highlight elm %}
     main =
       Html.program.
           { init = init
           , update = update
           , subscriptions = subscriptions
           , view = view
           }
    {% endhighlight %}

All of this took me about a day to figure out.  I hope this saves you some time.

Thanks!
- Matt

{% include email.html %}


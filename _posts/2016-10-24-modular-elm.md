---
title: Modular Elm
layout: post
---

Objective: Write an app that uses Model-View-Update but is not a giant Main module.

Caveat: This might not be the best way to structure an Elm app.
It's just what I did.

## Main

What is the role of Main?
Main, in this architecture, is aggregates submodules.
It has no (business) logic of its own.
This'll make more sense below.

## The Model

The elm example app in the docs starts with this trivial model:

    {% highlight elm %}
    type alias Model = Int
    {% endhighlight %}

We could move this to a counter module and change `Main.Model` to a record like this:

    {% highlight elm %}
    type alias Model =
        { counter = Counter.Model
        }
    {% endhighlight %}

## The Msg Type

The model was easy.
The update and the view are more complicated.
We need a way to map the module's `Msg` type into something that `Main` understands.
In the example, `Main.Msg` is defined as:

    {% highlight elm %}
    type Msg = Increment | Decrement
    {% endhighlight %}

Start by moving this to the counter module.

Next, we need to update Main.Msg to wrap this type.

    {% highlight elm %}
    type Msg = CounterMsg Counter.Msg
    {% endhighlight %}

Easy enough.
So how does this change `Main.update`?

## The update

Instead of processing the update directly, we need to delegate it to the module:

    {% highlight elm %}
    update msg model =
        case msg of
            CounterMsg counterMsg ->
                let
                    (newCounterModel, counterCmd) = Counter.update counterMsg model.counter
                in
                    ( { model | counter = newCounterModel }
                    , Cmd.map CounterMsg counterCmd
                    )

    {% endhighlight %}

There's a lot going on here.
Let's break it down.

1. Use `case` to pattern match on `msg` like we normally would.
2. Pass `model.counter` and `counterMsg` to `Counter.update`.
We're simply unwrapping the wrapped values and passing them to module's `update`.
3. Update `Main`'s counter with the new model value we get back.
4. *Important*.  Use `Cmd.map` to wrap any `Cmd` values the `update` function might return.
This gets it back into a type of `Msg` that `Main` understands.

## The view

The view follows a similar, but simpler, pattern.
To delegate to a view in a module, simply extract the html into a function and place it in the module.

    {% highlight elm %}
    view model =
        div []
            [ Html.map CounterMsg (Counter.view model.counter) ]
    {% endhighlight %}

Note that you need to use `Html.map` to wrap `Counter.view`'s return value.
This is because `Counter.view` has return type `Html Counter.Msg`, but `Main.view` only understands `Main.Msg` values.
Applying `Html.map` with an approprate `Main.Msg` constructor fixes this problem.

## Subscriptions

Finally, we need to delegate events from subscriptions.
This, again, follows the same pattern.

    {% highlight elm %}
    subscriptions : Model -> Sub Msg
    subscriptions model =
        Sub.batch
            [ Sub.map CounterMsg (Ports.somePort Counter.handleSomePort)
            ]
    {% endhighlight %}

Here, port just refers to some JavaScript interop code you may or may not have.
See my [elm-interop](/2016/10/14/elm-interop) post for more info.

## Conclusion

This is just one way to solve the problem.
The downside is that there is a decent amount of boilerplate.
The upside, though, is that there is no magic.

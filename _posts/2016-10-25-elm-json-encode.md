---
title: Encoding JSON with Elm
layout: post
---

If you Google Elm and Json, you'll find a few good articles.
For example, this one from Thoughtbot: [Decoding JSON Structures with Elm](https://robots.thoughtbot.com/decoding-json-structures-with-elm).
You'll notice, though, that they're all about decoding JSON data into Elm data structures.
There's not a whole lot about generating JSON from ELM data.

## The easy way

There's a reason for this.
In the simple case, Elm can automatically convert a record into a JSON object.
However, this breaks down if you have some oddly shaped JSON.
For example: The AWS JavaScript library likes to PascalCase its key names rather than camelCase them.
If you try to create an Elm record using PascalCase, you'll get a syntax error.


    {% highlight elm %}
    I ran into something unexpected when parsing your code!

    10|     , Foo : String
              ^
    I am looking for one of the following things:

        a lower case name
        whitespace
    {% endhighlight %}

When this happens, you'll need to encode the JSON yourself.

## Encoding with Json.Encode

The Json.Encode library provides all the primitives you need to generate JSON.
Next, you need some basic patterns for using the library effectively.
I've used 2.

1. Use union types to describe your data.
This lets you define a single polymorphic toJson function.
The downside is that I've found it difficult to understand.
2. Define functions for emitting JSON.
This is more familiar; its basically how HTML generation works.
You'll lack a single type definition for your JSON data model though.

My preference, right now, is #2.

### JSON generating functions

Assume we're tying to generate some JSON that looks like this:

    {% highlight javascript %}
    {
        "Foo": {
            "Bar": "Baz"
        }
    }
    {% endhighlight %}

The input data on the Elm side looks like this:

    {% highlight elm %}
    type alias Foo =
        { foo : Bar
        }
    type alias Bar =
        { bar : Baz
        }
    type Baz = Baz | Quux
    {% endhighlight %}

In general, the functions to do this will look like this:

    {% highlight elm %}
    toJson : SomeElmType -> Json.Encode.Value
    {% endhighlight %}

So, to go from our Elm type Foo to JSON, we'll define these functions:

    {% highlight elm %}
    fooJson : Foo -> Value
    fooJson foo =
        object [ ("Foo", barJson foo.foo) ]
    barJson : Bar -> Value
    barJson bar =
        object [ ("Bar", bazJson bar.bar) ]
    bazJson : Baz -> Value
    bazJson baz =
        case baz of
            Baz -> string "Baz"
            Quux -> string "Quux"
    {% highlight elm %}

As I mentioned yesterday, this solution has some unfortunate boilerplate, but it has no magic.

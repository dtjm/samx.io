+++
Tags = [ "golang" ]
menu = "main"
date = "2016-11-06T21:51:21-08:00"
title = "Designing Go libraries: Avoid package-level state"
draft = false
+++

Maintaining state in software programs sucks, but it is often a necessary
**evil**.  

*Global* state is terrible. 

But global state that is *mutable* (read/write) is THE WORST kind of state.

In Go programs, package-level state is a form of global state. If you must have
state, keep it contained to as small a portion of time and space as possible.

## How do I know if I have package-level state?

If you have a `var` declaration in the [top
level](https://github.com/afex/hystrix-go/blob/39520ddd07a9d9a071d615f7476798659f5a3b89/hystrix/circuit.go#L24-L27)
of your package (i.e. outside of a function or method definition), then you have
package-level state. This state is shared among all the code in your package.

If you have a `const` declaration, you are in good shape because this is
*immutable*, not mutable state (i.e. the value cannot be modified).

Having package-level state prevents your user from doing cool things like:

1. Reasoning about your library code (that is in the expected state at any given
   time)
2. Parallelizing tests

Sometimes you want to have a package level instance in your library, to make it
more convenient for your users (i.e. so they can use your library out of the
box, without any instantiation.) If you decide to take this route, please — for
the love of Gophers — provide a way to instantiate a single instance. Some
good examples of this are the [log.New](https://golang.org/pkg/log/) and
[flag.NewFlagSet](https://golang.org/pkg/flag/) methods from the standard
library. 
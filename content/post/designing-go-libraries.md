+++
Description = ""
Tags = [ "golang" ]
menu = "main"
date = "2016-11-06T21:51:21-08:00"
title = "Designing libraries in Go"
+++

I am not a Go expert, but in the past few years I've collected some ideas of
what makes a Go library easy to use. Here are a few humble suggestions I have
for you, the aspiring Go library designer, which you can use to bring joy and
delight to your users.

## Keep it primitive

The more of your library's surface area is tightly coupled to your library, the
more trouble your users will have if they want to make it cooperate with the
other libraries in their toolbox.

As a rule of thumb, your library should prefer to pass data to and from your
user in terms of primitive data types, if it makes sense.

If it doesn't make sense to do so, you may be tempted to create a struct to
organize the data that goes between your code and your user's code.

## Avoid package-level state

In general, maintaining state is a necessary **evil**. Global state is the worst
kind of state, and package-level state is a form of global state. If you must
have state, keep it contained to as short a time period as possible.

*How do I know if I have global state in my library?*

If you have a `var` declaration in the [top
level](https://github.com/afex/hystrix-go/blob/39520ddd07a9d9a071d615f7476798659f5a3b89/hystrix/circuit.go#L24-L27)
of your package (i.e. outside of a function or method definition), then you have
global state. This state is shared among all the code defined in
your library.

Having package-level state prevents your user from doing cool things like:

1. Reasoning about your library code (that is in the expected state at any given
   time)
2. Parallelizing tests

Sometimes you want to have a package level instance in your library, to make it
more convenient for your users (i.e. so they can use your library out of the
box, without any instantiation.) If you decide to take this route, please, for
the love of Gophers, also provide a way to instantiate a single instance. Some
good examples of this are the [log](https://golang.org/pkg/log/) and
[flag](https://golang.org/pkg/flag/) packages from the standard library.

## Don't parse configuration in your library

I've seen this done before and it is a recipe for trouble. When you inevitably
have a misconfiguration issue, your user has one more place to track down where
the config is being parsed. Let your user's code be responsible for parsing
configuration, and passing it down to your library. Maybe provide some helpers
functions to do this.

## Embed your dependencies

There's nothing more annoying in Go than **package management**.  Don't make your
users do extra and unnecessary package management.  Your library should come as a
single unit wrapped in a nice, neat bow.

Copy-pasting your dependencies directly into your package adds a little more
work to your plate, but makes working with your library slightly more
delightful.  You could save a poor Go newbie from dependency hell.

## Minimize interface surface area

Sometimes you want to abstract away the implementation of a component in your
library, for example a logger.  

So you think to yourself, "I'm such a nice person, I'm going to let my user pass
in any logger of their choosing".  Let them give me any `Logger` they want:

```
type Logger interface {
	Print(v ...interface{})
	Printf(format string, args ...interface{})
	Println(v ...interface{})
}
```

Stop right there. What you're doing there is **lazy**. You've destined your user
to do one of two things: 1) use the standard library
[log](https://golang.org/pkg/log/) package, or 2) write an adapter which has 3
methods. That is not something a nice person would do.

The astute reader may notice that some of these methods can be expressed in
terms of the others. In other words, why require `Println` when you already have
`Printf`? One can be expressed in terms of the other — for example:

```
func Println(s string) {
	Printf("%s\n", s)
}
```

This gives us the opportunity to minimize the interface we require. Let's try
this:

```
type Logger interface {
	Printf(format string, args ...interface{})
}
```  

That's a little better, right?

Now, let's say we wanted to plug in the
[(*testing.T).Logf](https://golang.org/pkg/testing/#T.Logf) method into the
`Logger` interface. We might create a wrapper like this:


```
type testLogger struct {
	*testing.T
}

func (t *testLogger) Printf(format string, v ...interface{}) {
	t.T.Logf(format, v...)
}
```

But if we made our library's logger interface even more abstract, we could get
away with even less code.

What's more abstract than an interface? How about a function?

```
type Logger func (format string, v ...interface{})
```

Exposing this behavior as a function instead of an interface makes it more
generic, and thus broadly compatible. With interfaces, you have to match the method
name, but with functions you only have to match the function signature, i.e. the
parameter and return types.

This, combined with Go's concept of [method
values](https://golang.org/doc/go1.1#method_values) gives us the ability to use
different loggers easily:

```
type Logger func (format string, v ...interface{})

var log Logger
var t *testing.T

// All of these loggers can be used
log = log.Printf
log = t.Logf
log = logrus.Debugf
log = glog.Infof
log = seelog.Debugf

log("I got an error: %s", err)
```

## Acknowledgments

Credit goes to these libraries for inspiring me:

- [Shopify/sarama](https://github.com/Shopify/sarama/blob/482c471fbf73dc2ac66945187f811581f008c24a/sarama.go#L61-L65)
- [google.golang.org/grpc](https://github.com/grpc/grpc-go/blob/e59af7a0a8bf571556b40c3f871dbc4298f77693/grpclog/logger.go#L50-L57)
- [afex/hystrix-go](https://github.com/afex/hystrix-go/blob/39520ddd07a9d9a071d615f7476798659f5a3b89/hystrix/circuit.go#L24-L27)
+++
Tags = [ "golang" ]
menu = "main"
date = "2016-12-16"
title = "Designing Go libraries: Minimize interface surface area"
draft = true
+++

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
generic, and thus broadly compatible. With interfaces, you have to match the
method name and signature, but with functions you only have to match the
signature, i.e. the parameters and return types.

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
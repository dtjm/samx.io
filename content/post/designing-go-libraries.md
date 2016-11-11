+++
Description = ""
Tags = [
  "Development",
  "golang",
]
Categories = [
  "Development",
  "GoLang",
]
menu = "main"
date = "2016-11-06T21:51:21-08:00"
title = "Designing libraries in Go"
+++

I am by far an expert in Go, but in the past few years I've collected some ideas
of what makes a Go library easy to use. Here are a few humble suggestions I have
for you, the Go library designer, which can bring joy and delight to your users.

## Minimize interfaces you accept

Sometimes you want to define the interface of something that gets passed into
your library, for example a logger.

So you think to yourself, "I'm such a nice guy, I'm going to let my user pass
in any logger of their choosing":

```
type Logger interface {
	Fatal(args ...interface{})
	Fatalf(format string, args ...interface{})
	Fatalln(args ...interface{})
	Print(args ...interface{})
	Printf(format string, args ...interface{})
	Println(args ...interface{})
}
```

Great! But, at the same time, not so great. You've destined your user to either use 
the standard library [log] package, or write an adapter which has 6 methods.

There is a better way. First of all, notice that some of these methods can be
expressed in terms of the others. In other words, why write `Println` when you
already have `Printf`? One is expressable in terms of the other.

```
func Println(s string) {
	Printf("%s\n", s)
}
```

This gives us the opportunity to minimize the interface we require:

```
type Logger interface {
	Fatalf(format string, args ...interface{})
	Printf(format string, args ...interface{})
}
```  

That's a little better, right?

Don't do [this](https://github.com/Shopify/sarama/blob/482c471fbf73dc2ac66945187f811581f008c24a/sarama.go#L61-L65), [that](https://github.com/Shopify/sarama/blob/master/mockresponses.go#L9-L14), or especially not [this](https://github.com/grpc/grpc-go/blob/master/grpclog/logger.go#L50-L57)

## 2a. Take a logger as a function
If you can reduce your 

## 1. Expose interfaces, not structs


## 3. Don't put state in the package

## 4. Embed your dependencies


```golang
// here is some code
```
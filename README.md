# Language design for `rat` #

`rat` is designed to add generic support in `go`. It uses the same syntax as `go`. So gophers should be able to read and use `rat` once they see it.

This repository contains documents related `rat` language design. If there is any language change, one should open issue in this repo. Issue will be closed by a change to some language spec files.

## Motivation ##

There are a lots of threads talking about generics in `go` on gopher-nuts mailing list. Response from go authors is always that go won't have generics.

Personally, I don't want to add generic to `go` either. This feature will significantly slow down `go` build speed. Tracking and instantiate generic types is quite expensive. As long as build speed is a goal, `go` will not have generic or similar features.

However, nothing can stop me to create a new language using `go` syntax with generic support. It will be useful for people who decide to sacrifice build time for writing less code. So this is why I start to write `rat`.

`rat` should be a super set of `go`. It means `rat` should include all go language features, even `go` 2.x features which may not have been designed yet. So `rat` will keep full compatibility at token level. There should never be new lexical elements out of [The Go Programming Language Specification](http://golang.org/ref/spec).

`rat` should focus on adding generic related features to `go`, not adding every go-abandoned features in it. A simple principle is that `rat` should only include necessary features to create a generic `Map` type. One should be able to use such `Map` just like a built-in map.

Here is a brief sample.

```go
// a generic Map type instantiation.
type MapInt struct {
  generic string, int
  Map
}

m := new(MapInt)

// operator overloading.
m["foo"] = 1

// getting and setting an item in map are different.
if _, ok := m["bar"]; ok {
  // ...
}

// another kind of operator overloading.
delete(m, "bar")

// generator support.
for item := range list {
  // ...
}
```

Any language feature that is not necessary for this map sample may be rejected as out of scope.

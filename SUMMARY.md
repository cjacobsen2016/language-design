# About `Rat` #

`Rat` is a general purpose language on top of `go`. In short, `rat` is a superset of `go` plus generic and generator. `Rat` uses standard `go` syntax to express generic and generator. All `go` features are available in `rat` without any modification.

This repository contains documents related `rat` language design. If there is any language change, one should open issue in this repository. Issue will be closed by a change to some language spec files.

## Design Principles ##

`Rat` treats `go` as a kind of asm. All `rat` files can be compiled to `go` source files.

The main design goal of `rat` is to allow developers to implement generic containers. Writing container library in `go` is quite painful. One has to use `interface{}` to store all data types, or write lots of similar structs/functions for all necessary types. `Rat` will define and implement a compile time generic like C++ template. One can write code once for all types.

The main goal is the only goal. All others are non-goals. Adding new features to a newly designed language is quite easy. Let us make it hard at begining. `Rat` should always be lean and focused.

To be more specific, if a feature is planned in `rat`, it must be a reasonable necessary part to implement following generic `Map`.

(Note: Syntax in the demo may change in future.)

```go
type Map struct { /* Suppose we have defined a generic Map. */ }

// Make a new map and provide some arguments for construction.
// Can be instantiated as an expression.
// Handle built-in function `make` and add some logic after made.
m := make(generic(Map, string, int), 10)

// Set item by operator `[]`.
m["foo"] = 12

// Get item by operator `[]`.
_ := m["foo"]

// Return two values when getting an item.
_, ok := m["foo"]

// Handler built-in function `delete`.
delete(m, "foo")

// Iterate over Map elements.
for k, v := range m {
	// ...
}

// Generic struct can have methods.
m.Clear()
```

## Motivation ##

There are a lot of threads talking about generics in `go` on gopher-nuts mailing list. Response from go authors is always that go won't have generics.

Personally, I don't want to add generic to `go` either. This feature will significantly slow down `go` build speed. Tracking and instantiate generic types is quite expensive. As long as build speed is a goal, `go` will not have generic or similar features.

However, nothing can stop me to create a new language using `go` syntax with generic support. It will be useful for people who decide to sacrifice build time for writing less code. So this is why I start to write `rat`.

## License ##

All code is licensed under MIT license that can be found in the LICENSE file.

All document work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).

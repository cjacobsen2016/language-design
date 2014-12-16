# Language design for `rat` #

`rat` is designed to add generic support in `go`. It uses the same syntax as `go`. So gophers should be able to read and use `rat` once they see it.

This repository contains documents related `rat` language design. If there is any language change, one should open issue in this repo. Issue will be closed by a change to some language spec files.

## Table Of Content ##

* [Language Design Principles](https://github.com/go-rat/language-design/blob/master/principles.md)
* [Language Specification](https://github.com/go-rat/language-design/blob/master/spec.md)
* Code Samples
  * [Generic Type](https://github.com/go-rat/language-design/blob/master/sample_generic_type.md)
  * [Generic Type Instantiation](https://github.com/go-rat/language-design/blob/master/sample_generic_instantiation.md)
  * [Built-in Function Overload](https://github.com/go-rat/language-design/blob/master/sample_overload.md)
  * [Generator](https://github.com/go-rat/language-design/blob/master/sample_generator.md)

## Motivation ##

There are a lot of threads talking about generics in `go` on gopher-nuts mailing list. Response from go authors is always that go won't have generics.

Personally, I don't want to add generic to `go` either. This feature will significantly slow down `go` build speed. Tracking and instantiate generic types is quite expensive. As long as build speed is a goal, `go` will not have generic or similar features.

However, nothing can stop me to create a new language using `go` syntax with generic support. It will be useful for people who decide to sacrifice build time for writing less code. So this is why I start to write `rat`.

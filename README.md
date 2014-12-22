# The Rat Programming Language Specification #

`Rat` is a general purpose language on top of `go`. In short, `rat` is a superset of `go` plus generic and generator. `Rat` uses standard `go` syntax to express generic and generator. All `go` features are available in `rat` without any modification.

`Rat`'s main goal is to enable developers to implement generic containers for `go`. `Rat` introduces a new kind of variable, the "type variable", to achieve this goal. One can define a type variable and use it in method and function just like a value variable. It's the `rat` way to implement generic.

`Rat` addes two new predeclared identifiers: `generic` and `yield`. Identifier `generic` is the key of type variable. Identifier `yield` is mainly used in generator functions.

There is no lexical change between `rat` and `go`. Standard package like `go/scanner` can process `rat` source file without any problem.

This repository defines all language features of `rat`. All language changes should be tracked and discussed in [project issues](https://github.com/go-rat/language-design/issues).

This specification can be viewed on gitbook: [The Rat Programming Language Specification](http://huandu.gitbooks.io/rat/).

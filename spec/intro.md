# Introduction #

This is a reference manual for the `rat` programming language.

`Rat` is a general purpose language on top of `go`. It derives all syntax from `go` and includes all `go` features without any modification. It's a superset of `go`.

`Rat`'s main goal is to enable developers to implement generic containers for `go`. `Rat` introduces a new kind of variable, the "type variable", to achieve this goal. One can define a type variable and use it in method and function just like a value variable.

`Rat` addes two new predeclared identifiers: `generic` and `yield`. Identifier `generic` is the key of type variable. Identifier `yield` is mainly used in generator functions.

There is no lexical change between `rat` and `go`. Standard package like `go/scanner` can process `rat` source file without any problem.

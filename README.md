# The Rat Programming Language Specification #

Rat is a general purpose language on top of [Go](https://golang.org). It's a superset of Go plus generic and generator.

All Go syntax is valid in Rat. This specification only defines syntax introduced by Rat. One should have a basic understanding of [The Go Programming Language Specification](http://golang.org/ref/spec) before reading this specification.

Rat's main goal is to enable developers to implement generic containers for Go. Rat introduces a new kind of variable, the "type variable", to achieve this goal. One can define a type variable and use it in any statement just like a value variable. It's the Rat way to implement generic.

Rat addes two new predeclared identifiers: `generic` and `yield`. Identifier `generic` is the key of type variable. Identifier `yield` is mainly used in generator functions.

[The "language-design" repository](https://github.com/go-rat/language-design) defines all language features of Rat. All language changes should be tracked and discussed in [project issues](https://github.com/go-rat/language-design/issues).

This specification can be viewed on [gitbook](http://huandu.gitbooks.io/rat/) for better reading experience.

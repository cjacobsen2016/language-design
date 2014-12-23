# The Rat Programming Language Specification #

Rat is a general purpose language on top of [Go](https://golang.org). It's a superset of Go plus generic and generator.

All Go syntax is valid in Rat. This specification only defines syntax introduced by Rat. One should have a basic understanding of [The Go Programming Language Specification](http://golang.org/ref/spec) before reading this specification.

Rat's design goal is to enable developers to implement generic data structures without repeating themselves. Rat introduces some new concepts, including type variable, generator and overriding, to achieve this gaol.

[The "language-design" repository](https://github.com/go-rat/language-design) defines all language features of Rat. All language changes should be tracked and discussed in [project issues](https://github.com/go-rat/language-design/issues).

This specification can be viewed on [gitbook](http://huandu.gitbooks.io/rat/) for better reading experience.

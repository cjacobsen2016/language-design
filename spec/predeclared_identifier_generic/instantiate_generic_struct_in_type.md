## Instantiate generic struct in type ##

The `generic` function is an expression rather than a type. It cannot be used in type declaration, especially field declaration in a struct.

If one wants to instantiate a generic struct in a struct, one can declare an anonymous struct and set its first field to `generic S` to do so.

```go
type S struct {
	T generic
}

type W struct {
	s struct {
		// This must be the first field declaration in this anonymous struct.
		generic S

		// List all type variables in S and set type for them.
		T int
	}
}
```

This syntax is included in spec for some rare cases. It's not recommended. In most cases, one should use `generic` function to define a new instantiated type variable outside struct declaration and use the defined variable in struct later.

```go
type S struct {
	T generic
}
SInt := generic(S, int)

type W struct {
	s SInt
}
```
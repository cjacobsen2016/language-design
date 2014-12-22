## Predeclared identifier `generic` ##

The predeclared identifier `generic` is the key of generic.

One can use it to declare type variable.

```go
// T is a type variable.
var T generic

type S struct {
	T generic // T is a type variable valid in struct context.
}
```

One can use it as a function to find out type of a value or define a new type variable based on a type variable.

```go
// Get type of 1.
T := generic(1) // T is int.

// Get a new instance of T.
T1 := generic(T) // The same as `type T1 T`. So `T1 !== T`.

// Get a map type.
T2 := generic(map[string]int) // The same as `type T2 map[string]int`.

// Instantiate a generic struct.
type S struct {
	T generic
}
SInt := generic(S, int)

// Instantiate a generic interface.
type I interface {
	generic(T)
	F(T)
}
IBool := generic(I, bool)
```

One can use it to instantiate a generic struct in type.

```go
type S struct {
	T generic
}

type SInt struct {
	generic S   // Instantiate S with following types.
	            // It must be the first declaration in struct.
	T       int
}
```

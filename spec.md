# The Rat Programming Language Specification #

## Introduction ##

This is a reference manual for the `rat` programming language.

`Rat` is a general purpose language on top of `go`. It derives all lexical elements in `go`. It means the standard package `go/scanner` can process `rat` source file without any problem. It also means a `go` file is a valid `rat` file without any modification.

## Language changes comparing with `go` ##

The major difference between `rat` and `go` is the concept of "type variable". Type becomes first-class citizen in `rat`. They can be assigned, stored, transferred and returned like a variable.

There are two new predeclared identifier in `rat`: `generic` and `yield`. Identifier `generic` is the key of type variable and generic. Identifier `yield` is the key of generator functions.

There is no lexical change between `rat` and `go`. All valid `go` source files are valid `rat` files without modification, unless a `go` file uses `rat` only predeclared identifier.

## Type variable ##

Type variable is a variable represent a type. Type name of any type is `generic`.

```go
// Declare a type variable. The type is generic.
var T generic

// T is int now.
T = int

i := T(1.2) // The same as int(1.2).

// T is float64 now.
T = float64

j := T(1) // The same as float64(1)

// Use type variable as a type.
// Declare a float64.
var k T
```

Type variable can store any valid type. On the other hand, any type, including predeclared types, can be used as a type variable. Keyword `type` is the `new` function for type variable.

Thinking of how to use `new` in `go`.

```go
s1 := new(S)
s2 := s1
s3 := new(S)

s1 == s2 // true
s1 == s3 // false
s2 == s3 // false
```

`s1` is a pointer to a value of `S`. `s2` points to the same memory as `s1`.

Back to type variable. Keyword `type` creates new type variable "value". Type variable is a "pointer" to the type "value". Specifically, type variable created by `type` is a constant variable.

```go
type Int int
T := int

// Int is not int. It's a new type derived
T == Int // false

Int = int // Compiler error. Int is a constant.
```

Type variable can be used in any expression, including assignment, function signature, struct field declaration, etc.

```go
// Use type variable and value variable in one expression.
T, i := int, 0

// Use type variable in function signature.
func F(T generic, i int) (S generic, ok bool)

// Use type variable in struct.
type S struct {
	T     generic
	Value S.T     // Use T in struct context.
}

// Use type variable in type assertion.
T := int
var i interface{} = 1
i.(T) // Get an int value.
```

### Type locking ###

Type variable can be modified in current block. If it's referred in other block, it will be locked for modification in most cases. Once it's locked, the referred one can be considered as an immutable copy of the original type variable.

```go
// Use type variable in closure.
func F() {
	T := int
	f := func() T {
		// T is locked and immutable. It's int.
		return T(1)
	}

	f()         // Returns int(1)
	T = float64
	f()         // Still returns int(1)
}

func M(T generic) {
	T = int // Compiler error. T is referred in closure by signature. It's immutable.
}

func N(i int) {
	T := int

	if i == 0 {
		T = float64 // Compiler error. T is locked and immutable.
	}
}
```

Due to type locking, it's always meanless to declare a "zero" type variable in global scope. So it's a Compiler error if one tries to do this.

```go
// Declare a "zero" type in global scope will trigger a Compiler error.
var T generic
```

There is one exception in type locking rule. When a block can be logically inlined to outer block at compile time, type variable declared in outer block can be modified in this block. Specifically, only plain block and block of `if` statement with constant logic expression can be applied to this exceptional rule.

```go
const N = 1

func K(i int) generic {
	T := int

	{
		T = int8 // Valid as the block can be inlined.
	}

	if T == float64 {
		return float32 // Valid. T == float64 is a compile time constant.
	}

	if N == 1 {
		return uint8 // Valid. N == 1 is a compile time constant.
	}

	if i == 0 {
		T = int16 // Compiler error. This if logic cannot be determined at compile time.
	}

	if i == 0 {
		return T // Compiler error.
		         // Implicitly modifies return value which is defined in outer scope.
	}

	return float64
}
```

### Lifespan ###

Type variable is only valid in compile phase. It cannot be accessed at runtime.

It's not possible to create an array at runtime to hold type variables, nor change any type variable based on a runtime value.

### Operators ###

Only comparison operators `==` and `!=` can be used with type variable. Type variables references the same type is equal.

```go
// `type` constructs new type. Int and int are different.
type Int int

T1 := Int
T2 := int
T3 := T1
T4 := T2

T1 == Int // true
T1 == int // false
T1 == T2  // false
T3 == T1  // true
T4 == int // true
```

## Generic struct ##

Generic struct is a struct with one or more type variables in field declaration. Generic struct cannot be constructed before instantiation.

```go
// S has a type variable T. It's a generic struct.
type S struct {
	T generic
	V T
}

// One cannot construct value with S directly.
v1 := S{} // Compiler error.

// Generic struct can be instantiated by `generic` function.
SInt := generic(S, int)
v2   := SInt{
	T: 1,
}
```

### Generic struct method ###

TBD

### Derive type variable ###

TBD

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

### Declare type variable ###

When a variable's type is `generic`, it's a type of type.

```go
// Following are valid declarations.

var T1 generic
T2 := generic(1)
func (T3 generic) (T4, T5 generic)
func (T6 ...generic)
```

However, `generic` itself is not a type variable. It's not possible to use `generic` with array, slice, `map` or `chan`, as all of them requires type variable in declaration.

```go
// One cannot use `generic` in this way.
var v1 [1]generic      // Compiler error.
var v2 map[generic]int // Compiler error.
var v3 map[int]generic // Compiler error.
var v4 chan generic    // Compiler error.
```

### `generic` function ###

One can use `generic` as a function. It takes any variable as parameter.

* If a value variable is provided, `generic` will return type of the variable.
* If a type variable is provided, `generic` will work as `type` and define new type variable.
* If a type variable is a generic struct, `generic` can take additional parameters to instantiate the generic struct.

Blank identifier `_` can be used in `generic` function to indicate one wants to partially instantiate a generic struct.

```go
// Get type of a value variable.
var v1 int
T1 := generic(v1) // T1 === int

// Define new type variable.
T2 := float64
T3 := generic(T2)
T2 === T3 // false

// Instantiate generic struct.
type S struct {
	T1, T2 generic
}
T4  := generic(S, int, float64) // Instantiate S while `S.T1 = int` and `S.T2 = float64`.
T5  := generic(S)               // Define a new type variable based on S.
T7  := generic(S, _, int)       // Partial instantiate S. S.T1 is still undefined.
T8  := generic(T7, float64, _)  // Fully instantiate S.
T9  := generic(S, bool)         // Compiler error. S needs two type variables.
T10 := generic(T7, float64)     // Compiler error. T7 still needs two type variables.
```

One cannot use `generic` function on built-in functions, blank identifier and `generic` itself.

```go
// One cannot use `generic` in this way.
T1  := generic(close)   // Compiler error.
T2  := generic(len)     // Compiler error.
T3  := generic(cap)     // Compiler error.
T4  := generic(new)     // Compiler error.
T5  := generic(make)    // Compiler error.
T6  := generic(delete)  // Compiler error.
T7  := generic(complex) // Compiler error.
T8  := generic(real)    // Compiler error.
T9  := generic(imag)    // Compiler error.
T10 := generic(panic)   // Compiler error.
T11 := generic(recover) // Compiler error.
T12 := generic(print)   // Compiler error.
T13 := generic(println) // Compiler error.
T14 := generic(_)       // Compiler error.
T15 := generic(generic) // Compiler error.
```

A special note on `generic` function. As `generic` function call is an expression, it's not allowed to use it in any syntax requires type.

Following are common errors one may make.

```go
// Don't do these.
v1 := generic(int)(0)              // Compile error.
v2 := generic(S, int){ /* ... */ } // Compile error.

// Do these instead.
T1 := generic(int)
T2 := generic(S, int)
v1 := T1(0)
v2 := T2{ /* ... */ }
```

### Instantiate generic struct in type ###

The `generic` function is an expression rather than a type. It cannot be used in type declaration, especially field declaration in a struct.

If a generic struct is required to be instantiated in a struct, one can declare an anonymous struct and set its first field to `generic S` to do so.

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

This syntax is included in spec for some rare cases. It's not recommended. In most cases, one should use `generic` function to define a new instantiated type variable outside struct declaration and use the variable in struct later.

```go
type S struct {
	T generic
}
SInt := generic(S, int)

type W struct {
	s SInt
}
```

## Generator function ##

TBD

## Overloading ##

### Built-in function overloading ###

TBD

* `make`
* `new`
* `delete`
* `range`
* `cap`
* `len`
* `close`

### Operator overloading ###

TBD

* operator `[]`
* operator `<-`

### Function overloading ###

TBD

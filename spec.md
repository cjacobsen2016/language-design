# The Rat Programming Language Specification #

## Introduction ##

This is a reference manual for the `rat` programming language.

`Rat` is a general purpose language on top of `go`. It derives all lexical elements in `go`. It means the standard package `go/scanner` can process `rat` source file without any problem. It also means a `go` file is a valid `rat` file without any modification.

## Language changes comparing with `go` ##

The major difference between `rat` and `go` is the concept of "type variable". Type becomes first-class citizen in `rat`. They can be assigned, stored, transferred and returned like a variable.

There are two new predeclared identifier in `rat`: `generic` and `yield`. Identifier `generic` is the key of type variable and generic. Identifier `yield` is the key of generator functions.

There is no lexical change between `rat` and `go`.

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

Type variable can store any valid type. In the other word, any type can be considered as a type variable. Keyword `type` is the `new` function for type variable.

Thinking of how to use `new` in `go`.

```go
s1 := new(S)
s2 := s1
s3 := new(S)

s1 == s2 // true
s1 == s3 // false
s2 == s3 // false
```

`s1` is a pointer to a value of `S`. `s2` points to the same memory as `s1`. There is only one `S` value in memory.

Back to type variable. Keyword `type` creates new type variable "value". Type variable is a "pointer" to the type "value". Specifically, type variable created by `type` is a constant variable.

```go
type Int int
T := int

// Int is not int. It's a new type derived
T == Int // false

Int = int // Compile error. Int is a constant.
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
	T = int // Compile error. T is referred in closure by signature. It's immutable.
}

func N(i int) {
	T := int

	if i == 0 {
		T = float64 // Compile error. T is locked and immutable.
	}
}
```

Due to type locking, it's always meanless to declare a "zero" type variable in global scope. So it's a compile error if one tries to do this.

```go
// Declare a "zero" type in global scope will trigger a compile error.
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
		T = int16 // Compile error. This if logic cannot be determined at compile time.
	}

	if i == 0 {
		return T // Compile error.
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

One can use it as a function to get type variable of any variable.

```go
// Get type of 1.
T := generic(1) // T is int.

// Get type of T. It's still T.
T1 := generic(T) // Same as T1 := T

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

When declaring a variable, `generic` works exactly the same as other predeclared identifier like `int`.

```go
// Following are valid declarations.

var T1 generic
T2 := generic(1)
func (T3 generic) (T4, T5 generic)
func (T6 ...generic)
```

### `generic` function ###

TBD

### Instantiate generic type ###

TBD

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

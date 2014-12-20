# The Rat Programming Language Specification #

## Introduction ##

This is a reference manual for the `rat` programming language.

`Rat` is a general purpose language on top of `go`. It derives all syntax from `go` and includes all `go` features without any modification. It's a superset of `go`.

`Rat`'s main goal is to enable developers to implement generic containers for `go`. `Rat` introduces a new kind of variable, the "type variable", to achieve this goal. One can define a type variable and use it in method and function just like a value variable.

`Rat` addes two new predeclared identifiers: `generic` and `yield`. Identifier `generic` is the key of type variable. Identifier `yield` is mainly used in generator functions.

There is no lexical change between `rat` and `go`. Standard package like `go/scanner` can process `rat` source file without any problem.

## Syntax ##

`Rat` has the same syntax as `go`. See [The Go Programming Language Specification](http://golang.org/ref/spec) for syntax details.

## Type variable ##

Type variable is a variable represent a type. Type name of a type is `generic`.

```go
// Declare a zero type variable.
var T generic

// T is int now.
T = int

i := T(1.2) // The same as int(1.2).

// T is float64 now.
T = float64

j := T(1) // The same as float64(1)

// Use type variable as a type.
// Declare a []float64.
var k []T = float64{1, 2}
```

### Usage ###

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

### Type variable and `type` keyword ###

Type variable can store any valid type. On the other hand, any type, including predeclared types, can be treated as a type variable. Any valid `go` type is a type variable and can be used as a variable.

Type variable can be considered as a pointer to underlying type. Operator `=` always copies pointer value, not the underlying type. The `type` keyword, on the other hand, always clones underlying type to define a new type.

```go
type T int
T1 := int
T2 := T

// T is not int. It's a new type based on int.
T  == int // false
T1 == T2  // false
T1 == int // true
T2 == T   // true
```

The type variable defined by `type`, the `T` in `type T int` for example, is immutable.

### Modify type variable ###

Type variable is mutable only in current code block and children blocks which can be logically inlined in compile-time. Specifically, type variable can be modified in a child block only if this block is a solely block or a `if` statement whose expression is a compile-time constant.

```go
const N = 1

func K(i int) generic {
	T := int

	// This is a solely block.
	{
		T = int8 // Valid as the block can be inlined.
	}

	if T == float64 {
		return float32 // Valid. `T == float64` is a compile-time constant.
	}

	if N == 1 {
		return uint8 // Valid. `N == 1` is a compile-time constant.
	}

	if i == 0 {
		T = int16 // Compile-time error. `i == 0` cannot be evaluated at compile-time.
	}

	if i == 0 {
		return T // Compile-time error.
		         // Implicitly modifies return value which is defined in outer scope.
	}

	return float64
}
```

When a type variable is immutable in a block, value of this variable is determined at the first time it's referred.

```go
// Use type variable in closure.
func F() {
	T := int
	f := func() T {
		// T is determined and immutable. It's int.
		return T(1)
	}

	f()         // Returns int(1)
	T = float64 // Changes T will not affect the T in f.
	f()         // Still returns int(1)
}

func M(T generic) {
	T = int // Compile-time error. T is defined in function signature. It's immutable.
}

func N(i int) {
	T := int

	if i == 0 {
		T = float64 // Compile-time error. T is locked and immutable.
	}
}
```

It's always meanless to declare a "zero" type variable in global scope. So it's a Compile-time error if one tries to do this.

```go
// Declare a "zero" type in global scope will trigger a Compile-time error.
var T generic
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

### Function/method signature ###

Type variable can be used as type in function or method signature. When defining a type variable in signature, it means function/method will accept a type variable as parameter.

```go
func F(T generic) {}
func (s *S) M(T generic) {}

// Provide type variable as parameter.
F(int)
s := &S{}
s.M(bool)
```

A defined type variable can be used as type in signature.

```go
func F(T generic, t T) {} // Valid.
func F(T generic) T {}    // Valid.
func G(t T, T generic) {} // Compile-time error. T must be defined before using it.
```

Use blank identifier `_` to indicate that a type variable can be deduced by other parameters. If compiler fails to deduce the type, it will raise a compile-time error.

```go
func F(T generic, t1, t2 T) {}
func G(T generic) G {}

F(_, 1, 2)   // Valid. as type of t1 and t2 is int, T is int.
F(_, 1, 2.0) // Compile-time error. Type of t1 is int but Type of t2 is float64. Type conflict.

G(_) // Compile-time error. No way to deduce T.
```

Variadic type variable parameter is supported. It means this function/method can take any number of type variables as parameters.

The variadic parameter is a very special variable. It's not a type variable. It looks like an array of type variables, but it's not allowed to construct slice from it. It can only be used to get type by index, test length through `len` and pass it to other variadic function with `...`.

```go
func F(types ...generic) {
	T1 := types[0]
	T2 := types[1]
	G(types...)
}
func G(types ...generic) {
	T1 := types    // Compile-time error.
	T2 := types[:] // Compile-time error.
}

F(int, bool, rune) // len(types) == 3
                   // T1 is int and T2 is bool.
F(float64)         // Compile-time error. T2 uses `types[1]` which is out of range.
```

## Generic struct ##

Generic struct is a struct with one or more type variables in field declaration.

### Instantiate generic struct ###

Generic struct must be instantiated explicitly or implicitly before constructing a value.

```go
// As S has a type variable T as a field, it's a generic struct.
type S struct {
	T generic
	V T
}

// One cannot construct S value without instantiation.
var v1 S                // Compile-time error.
v2   := S{}             // Compile-time error.
v3   := S{V: 1}         // Valid. Implicitly instantiate S with int.
v4   := S{T: int}       // Valid. Explicitly instantiate S with int.
SInt := generic(S, int) // Valid. Explicitly instantiate S with int.
                        // SInt is not a generic struct any more.
v5   := SInt{}          // Valid.
```

### Type variable field declaration ###

There is no special syntax for type variable field declaration. It's just a field whose type is `generic`.

Type variable field must have a unique and non-blank name within a struct. Type variable field cannot be anonymous.

```go
type S struct {
	T1, T2 generic             // Valid.
	T3     generic `foo:"bar"` // Valid.
	_      generic             // Compile-time error. Name cannot be blank identifier.
	T1     int                 // Compile-time error. Duplicated name `T1`.

	generic // Compile-time error. Generic itself is not a type.
	T3      // Compile-time error. Conflict name with `T3 generic`.
}
```

Type variable field can be read through a constructed struct value.

```go
type S struct {
	T generic
}

v1 := S{T: int}
v2 := &S{T: float32}
v3 := &v2

T1 := v1.T    // T1 is int.
T2 := v2.T    // T2 is float32.
T3 := (*v3).T // T3 is float32.
```

### Method declaration ###

Method declaration syntax is the same as that in `go`. Type of method parameters can be a type variable field.

```go
type Array struct {
	T generic
	data []T
}

// arr.T can be used as parameter type.
func (arr *Array) Append(values ...arr.T) {
	arr.data = append(arr.data, values...)
}

// arr.T can be used as return type.
func (arr *Array) Get(index int) arr.T {
	return arr.data[index]
}
```

### Field using generic struct ###

When type of a generic struct field is a generic struct, field type can use type variables defined in the struct by name to get instantiated implicitly. If field type cannot find a type variable with expected name, field type cannot be instantiated thus cannot be constructed.

```go
type List struct {
	ValueType  generic
	Head ListNode      // ListNode use the ValueType to instantiate itself.
}

type ListNode struct {
	ValueType  generic
	Prev, Next *ListNode
	Value      ValueType
}

list := &List{
	ValueType: int,
}
list.ValueType      == int // true
list.Head.ValueType == int // true

type WiredList struct {
	T    generic
	Head ListNode // Compile-time error. Cannot use ListNode without instantiation.
}

type SuperListNode struct {
	ValueType generic
	extra     interface{}
	ListNode              // Generic struct can be an anonymous field.
}
```

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

When a variable's type is `generic`, it's a type variable.

```go
// Following are valid declarations.

var T1 generic
T2 := generic(1)
func (T3 generic) (T4, T5 generic)
func (T6 ...generic)
```

However, `generic` itself is not a type variable. One cannot use `generic` to instantiate generic struct or define type with array, slice, `map` or `chan`.

```go
// One cannot use `generic` in this way.
var v1 []generic       // Compile-time error.
var v2 [1]generic      // Compile-time error.
var v3 map[generic]int // Compile-time error.
var v4 map[int]generic // Compile-time error.
var v5 chan generic    // Compile-time error.
```

Defining `generic` pointer makes no sense.

```go
var v1 *generic // Compile-time error.
```

### `generic` function ###

One can use `generic` as a function. It takes any variable as parameter.

* If a value variable is provided, `generic` will return type of the variable.
* If a type variable is provided, `generic` will work as `type` and define new type variable.
* If a type variable is a generic struct, `generic` can take additional parameters to instantiate the generic struct.
* Type of `nil` is a zero type.
* If no parameter is provided, return a zero type.

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
T9  := generic(S, bool)         // Compile-time error. S needs two type variables.
T10 := generic(T7, float64)     // Compile-time error. T7 still needs two type variables.

var T11 generic     // Zero type.
T12 := generic(nil) // Zero type.
T13 := generic()    // Zero type.
T11 == T12          // true
T11 == T13          // true
```

One cannot use `generic` function on built-in functions, blank identifier, variadic type variable parameter and `generic` itself.

```go
// One cannot use `generic` in this way.
T1  := generic(close)   // Compile-time error.
T2  := generic(len)     // Compile-time error.
T3  := generic(cap)     // Compile-time error.
T4  := generic(new)     // Compile-time error.
T5  := generic(make)    // Compile-time error.
T6  := generic(delete)  // Compile-time error.
T7  := generic(complex) // Compile-time error.
T8  := generic(real)    // Compile-time error.
T9  := generic(imag)    // Compile-time error.
T10 := generic(panic)   // Compile-time error.
T11 := generic(recover) // Compile-time error.
T12 := generic(print)   // Compile-time error.
T13 := generic(println) // Compile-time error.
T14 := generic(_)       // Compile-time error.
T15 := generic(generic) // Compile-time error.

func F(types ...generic) {
	T16 := generic(types) // Compile-time error.
}
```

A special note on `generic` function. As `generic` function call is an expression, it's not allowed to use it in any context when a type variable is required.

Following are common errors one may make.

```go
// Don't do these.
v1 := generic(int)(0)   // Compile error.
v2 := generic(S, int){} // Compile error.

// Do following instead.
T1 := generic(int)
T2 := generic(S, int)
v1 := T1(0)
v2 := T2{}
```

### Instantiate generic struct in type ###

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

## Generator ##

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

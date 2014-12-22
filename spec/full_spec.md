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
func F(T generic, i int) (S generic, ok bool) {}

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

## Generic interface ##

Generic interface is an interface contains a method with name `generic`. All types, not variables, in this `generic` method are type variables. They can be used in all methods of this interface.

```go
// A generic interface.
type I interface {
	generic(T)
	F(T)
}
```

Generic interface can be used to check whether a generic struct implements expected methods. It cannot be used in any type assertion as it requires runtime information.

```go
type I interface {
	generic(T)
	F(T)
}

// S1 implements I.
type S1 struct {}
func (s1 S1) F(v int) {}

// S2 also implements I.
type S2 struct {
	T2 generic
}
func (s2 S2) F(v s2.T2) {}

// S3 doesn't implement I, as S3#F takes 2 parameters instead of 1.
type S3 struct {
	T3 generic
}
func (s3 S3) F(T generic, v int) {}
```

When a generic interface is fully instantiated, it works exactly the same an non-generic interface.

```go
type I interface {
	generic(T)
	Add(T)
}

IInt := generic(I, int)

// S implements IInt.
type S struct {}
func (s S) Add(int) {}

var s S
s.(IInt) // OK.
```

If `generic` method has result, the result will be instantiated as method receiver's type. As there is only one receiver for a method, it's meanless to declare more than one type variables in result.

```go
type I interface {
	generic() Receiver
	New() Receiver
}

// T1 implements I.
type T1 int
func (t1 T1) New() T1

// T2 doesn't implement I.
// Return type doesn't match receiver type as T2 != int.
type T2 int
func (t2 T2) New() int {}
```

Type variable cannot be used in interface method other than `generic`.

```go
type I interface {
	InvalidM(T generic) // Compile-time error.
}
```

### Embedding generic interface ###

If a generic interface is embedded in another interface, all type variables defined in this interface must also be defined in the interface embedding it. So, only generic interface can embed generic interface.

```go
type I1 interface {
	generic(T)
	M(T)
}

// Valid.
type I2 interface {
	generic(T)
	I1
}

// Compile-time error. I3 doesn't have a type variable named T.
type I3 interface {
	generic(S)
	I1
}
```

### Variadic type variables in generic interface ###

The `generic` method in a generic interface can have any number of variadic type variables. Each of them can represent any number of parameters in a method. Once a generic interface is instantiated, variadic type variables are set to a specific list of parameters.

```go
type I interface {
	generic(...Args)

	// M1 and M2 must have the same parameters in type.
	M1(Args)
	M2(Args)
}

// T1 implements I.
type T1 int
func (t1 T1) M1(int, bool) {}
func (t1 T1) M2(int, bool) {}

// T2 doesn't implement I as parameters of M1 and M2 are different.
type T2 int
func (t2 T2) M1(int, bool) {}
func (t2 T2) M2(float64, string) {}
```

Mind that variadic type variable is not a type variable. It's not valid to use "..." in method with variadic type variable.

```go
type I interface {
	generic(...Args)

	// Compile-time error.
	// Args is not a type variable. Adding "..." doesn't make any sense.
	InvalidM(...Args)
}
```

Variadic type variables can appear in any position of parameters or result. Compiler will try its best to find a matchable pattern.

```go
type I interface {
	generic(...Args, ...Return)
	M1(int, Args) (Return, error)
}

// T implements I.
// Args is instantiated as (float64, string).
// Result is instantiated as (T, bool).
type T int
func (t T) M1(int, float64, string) (T, bool, error) {}
```

Two variadic type variables in generic interface cannot be used in one parameters or result declaration, as there is no way for compiler to decide how to instantiate these type variables.

```go
type I interface {
	generic(...Args1, ...Args2)

	// Compile-time error. Args1 and Args2 cannot be used in one parameters declaration.
	InvalidM(Args1, Args2)
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

### Declare type variable ###

When a variable's type is `generic`, it's a type variable.

```go
// Following are valid declarations.

var T1 generic
T2 := generic(1)
func (T3 generic) (T4, T5 generic) {}
func (T6 ...generic) {}
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
  * If the type variable is a generic struct, `generic` can take additional parameters to instantiate the generic struct.
  * If the type variable is a generic interface, `generic` can take additional parameters to instantiate the generic interface.
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

Generator is a special function or method which can be used to control the iteration behavior of a loop. When a function or method has only one result parameter in which identifier name is `yield` and type is a function, it's a generator.

```go
// Valid. Gen1 is a generator function.
func Gen2() (yield func(int)) {}

// Valid. Gen2 is a generator method.
func (s S) Gen2() (yield func(int)) {}

// Valid. Type of `yield` can be a function returning values.
func Gen3() (yield func(int) bool) {}

// Compile-time error. The identifier `yield` cannot be used elsewhere.
func F1(yield int) {}

// Compile-time error. Type of `yield` must be a function.
func F2() (yield int) {}

// Compile-time error. Only one parameter in result is allowed.
func F3() (yield func(int), s S) {}
func F4() (s S, yield func(int)) {}
```

Type of `yield` in result must be a function. This function cannot take any type variable in function signature, both parameters and result.

```go
// Compile-time error. Type of `yield` cannot take any type variable.
func F() (yield func(T generic)) {}
```

When caller calls a generator, the generator will return an iterator value. The returned iterator has `Next` method. Method signature is determined by `yield` type in generator. Specifically, `Next` method's result type is all `yield` parameters plus a bool indicating done; `Next` method parameters is `yield` result.

```go
func Gen() (yield func(int, float32) string) {}

iter := Gen()

// Next signature is func(string) (int, float32, bool).
iter.Next("foo")
```

Caller uses iterator value's `Next` method to get value from generator. On the other hand, generator uses `yield` as a function inside function body to generate value one by one. Once `yield` is called, generator will be halted until caller consumes the value. If generator returns using `return` statement or is panic, future calls to `Next` will always return zero yielded values plus a true, which means iteration is done.

```go
func Gen() (yield func(int)) {
	fmt.Println("Gen: before yield.")
	yield(1)
	fmt.Println("Gen: 1st yield is called.")
	yield(3)
	fmt.Println("Gen: 2nd yield is called.")
}

func main() {
	iter := Gen()

	// As `yield` type is func(int), type of Next method is func() (int, bool)
	fmt.Println("main: before first Next.")
	fmt.Println(iter.Next())
	fmt.Println("main: 1st Next is called.")
	fmt.Println(iter.Next())
	fmt.Println("main: 2nd Next is called.")
	fmt.Println(iter.Next())
	fmt.Println("main: 3rd Next is called.")
	// Output:
	// main: before first Next.
	// Gen: before yield.
	// 1	false
	// main: 1st Next is called.
	// Gen: 1st yield is called.
	// 3	false
	// main: 2nd Next is called.
	// Gen: 2nd yield is called.
	// 0	true
	// main: 3rd Next is called.
}
```

When `yield` is a function returning values, caller can use `Next` method to set values returned by `yield`. First call to `Next` method is special. A generator can only get returning values by calling `yield`. As code in generator doesn't start executing until `Next` is called for first time, there is no `yield` at begining thus no way to retrieve first `Next` parameters in generator. It's a by-design behavior.

```go
func Gen() (yield func() int) {
	fmt.Println("yield:", yield())
	fmt.Println("yield:", yield())
}

func main() {
	iter := Gen()
	iter.Next(1) // Parameters in first Next cannot be retrieved in generator.
	iter.Next(2)
	iter.Next(3)
	// Output:
	// yield: 2
	// yield: 3
}
```

Iterator value returned by generator can work with `range` expression. Type of iteration variables in `range` clause are determined by `yield` parameter types. Count of iterator variables must be not greater than that of `yield` parameters.

```go
func Gen() (yield func(int, float64)) {
	yield(1, 2.3)
	yield(2, 3.4)
}

func main() {
	// Output:
	// 1	2.3
	// 2	3.4
	for i, j := range Gen() {
		fmt.Println(i, j)
	}

	// Output:
	// 1
	// 2
	for i := range Gen() {
		fmt.Println(i)
	}

	// Output:
	// 2.3
	// 3.4
	for _, j := range Gen() {
		fmt.Println(j)
	}
}

func illegal() {
	// Compile-time error. Gen's `yield` only yields two values at most.
	for i, j, k := range Gen() {
		// ...
	}
}
```

All iterator values implement `Iterator` type. This type is defined in `github.com/go-rat/rat/types`.

```go
// Defined in "github.com/go-rat/rat/types".
type Iterator interface {
	generic(...Yield, ...Result)
	Next(Result) (Yield, bool)
}

func Gen() (yield func(int)) {}

// Valid.
var it Iterator = Gen()
it.Next()
```

## Overriding ##

Many `go` built-in functions and operators only work with built-in types like map, slice and chan. `Rat` extends them to work with custom types by overriding mechanism.

Override mechanism is based on interface types. Every built-function or operator, which can be overridden, has one special interface for overriding. If a type implements the interface, it can override the respective function or operator.

If one decides to use overriding feature, one should make sure semantics of overridden function or operator is not changed.

For example, built-in function `cap` expects a value implements `CapOverrider` interface, which is defined as following.

```go
type CapOverrider interface {
	OverrideCap() int
}
```

If a type implements `CapOverrider`, `CapOverrider` will be called when `cap` is called against type's value.

```go
type Array []int

func (arr Array) OverrideCap() int {
	return 10
}

var a Array
println(cap(a)) // Always prints 10.
```

All interfaces related to overriding are defined in `github.com/go-rat/rat/types`.

### Built-in function `new` ###

A type must implement `NewOverrider` to override `new`. Method receiver is initialized to zero value of receiver type.

```go
type NewOverrider interface {
	generic() Receiver
	OverrideNew() Receiver
}
```

Following is an example.

```go
type S struct {
	V int
}

// Override `new`.
func (s *S) OverrideNew() *S {
	// Receiver `s` is set to zero value.

	// Set s.V to a special value.
	return &S{
		V: 1,
	}
}

s := new(S)
s.V == 1 // true
```

### Built-in function `make` ###

A type must implement `MakeOverrider` to override `make`. Method receiver is initialized to zero value of receiver type.

```go
type MakeOverrider interface {
	generic(...Args) Receiver
	OverrideMake(Args) Receiver
}
```

Following is an example.

```go
type S struct {
	Data []int
}

// Override `make`.
func (s *S) OverrideMake(size, cap int) *S {
	// Receiver `s` is set to zero value.

	// Set s.V to a special value.
	return &S{
		Data: make(generic(S.Data), size, cap),
	}
}

s := make(S, 10, 20)
len(s.Data) == 10 // true
cap(s.Data) == 20 // true
```

### Built-in function `delete` ###

A type must implement `DeleteOverrider` to override `delete`. Method receiver is set to the first parameter of `delete`.

```go
type DeleteOverrider interface {
	generic(Index)
	OverrideDelete(Index)
}
```

Following is an example.

```go
type S struct {
	M map[string]int
}

// Override `delete`.
func (s *S) OverrideDelete(index string) {
	delete(s.M, index)
}

s := &S{}
s.M = make(generic(s.M)) // Make a map value.
s.M["foo"] = 123

delete(s, "foo")
s.M["foo"] == nil // true
```

### Built-in function `cap` ###

A type must implement `CapOverrider` to override `cap`. Method receiver is set to the first parameter of `cap`.

```go
type CapOverrider interface {
	OverrideCap() int
}
```

Following is an example.

```go
type T []int

// Override `cap`.
func (t T) OverrideCap() int {
	return 10
}

var t T
cap(t) == 10 // true
```

### Built-in function `len` ###

A type must implement `LenOverrider` to override `len`. Method receiver is set to the first parameter of `len`.

```go
type LenOverrider interface {
	OverrideLen() int
}
```

Following is an example.

```go
type T []int

// Override `len`.
func (t T) OverrideLen() int {
	return 10
}

var t T
len(t) == 10 // true
```

### Built-in function `close` ###

A type must implement `CloseOverrider` to override `close`. Method receiver is set to the first parameter of `close`.

```go
type CapOverrider interface {
	OverrideClose()
}
```

Following is an example.

```go
type T chan int

// Override `close`.
func (t T) OverrideClose() int {
	println("Closing...")

	// Cast T to a plain chan type to avoid recursive call.
	var c chan int = t
	close(c)
}

t := make(T)
close(t) // Output: Closing...
```

### Range expression ###

A type must implement `RangeOverrider` to work with `range` expression. Method receiver is set to the expression value in `range` clause.

```go
type RangeOverrider interface {
	generic(...Args)
	OverrideRange() (yield func(Args))
}
```

Following is an example.

```go
type S struct {
	Data []int
}

// Override `range` expression.
func (s *S) OverrideRange() (yield func(int, int)) {
	for i := range s.Data {
		yield(i, i * 2)
	}
}

s := &S{}
s.Data = []int{1, 2, 3, 4}

// Output:
// 1	2
// 2	4
// 3	6
// 4	8
for i, j := range s {
	fmt.Println(i, j)
}
```

### Index expression ###

A type must implement `IndexOverrider` to work with slice expression. Method receiver is set to the indexed value. When index expression is called in a double-value context, `OverrideGetItemOK` will be called. Otherwise, `OverrideGetItem` is called for expression and `OverrideSetItem` is called for assignment.

```go
type IndexOverride interface {
	generic(Index, Value)

	OverrideGetItem(Index) Value
	OverrideGetItemOK(Index) (Value, bool)
	OverrideSetItem(Index, Value)
}
```

Following is an example.

```go
type S struct {
	Data map[string]int
}

func (s *S) OverrideGetItem(index string) int {
	return s.Data[index] + 2
}

func (s *S) OverrideGetItemOK(index string) (int, bool) {
	v, ok := s.Data[index]

	if !ok {
		return 0, false
	}

	return v + 2, true
}

func (s *S) OverrideSetItem(index string, value int) {
	s.Data[index] = value + 2
}

s := &S{}
s["foo"] = 2
println(s["foo"]) // Prints: 6
_, ok := s["bar"]
println(ok)       // Prints: false
```

### Slice expression ###

A type must implement `SliceOverrider` to work with slice expression. Method receiver is set to the sliced value. When low is not set in expression, it will be 0 in method parameter. When high is not set in expression, it will be `len(value)` in method parameter.

```go
type SliceOverrider interface {
	generic() Receiver
	OverrideSlice(low, high int) Receiver
	OverrideSlice3(low, high, max int) Receiver
}
```

Following is an example.

```go
type S struct {
	Data []int
}

func (s *S) OverrideSlice(i, j int) *S {
	return &S{
		Data: s.Data[i:j],
	}
}

func (s *S) OverrideSlice(i, j, k int) *S {
	return &S{
		Data: s.Data[i:j:k],
	}
}

s := &S{}
s.Data = []int{1, 2, 3, 4, 5}
s1 := s[1:3]
println(s1.Data)      // Prints: [2 3]
s2 := s[1:3:4]
println(cap(s2.Data)) // Prints: 3
```

### Receiver operator ###

A type must implement `ReceiverOverrider` to override receiver operator. Method receiver is set to the value of operand. If receiver operator is called in a double-value context, `OverrideReceiverOK` will be called. Otherwise, `OverrideReceiver` is called.

```go
type ReceiverOverrider interface {
	generic(Value)
	OverrideReceiver() Value
	OverrideReceiverOK() (Value, bool)
}
```

Following is an example.

```go
type T int

func (t T) OverrideReceiver() int {
	i, ok := t.OverrideReceiverOK()

	if !ok {
		panic("Exhausted.")
	}

	return i
}

func (t T) OverrideReceiverOK() (int, bool) {
	if t <= 0 {
		return 0, false
	}

	t--
	return t, true
}

t := T(3)

println(<-t) // Prints: 2
println(<-t) // Prints: 1
println(<-t) // Prints: 0
println(<-t) // Panic.
```

### Send statement ###

A type must implement `SendOverrider` to override send statement. Method receiver is set to the channel expression.

```go
type SendOverrider interface {
	generic(Value)
	OverrideSend(Value)
}
```

Following is an example.

```go
type T int

func (t T) OverrideSend(value int) {
	t += T(value)
}

var t T

t <- 10
t <- 20

println(t) // Prints: 30
```

## `generic` function ##

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
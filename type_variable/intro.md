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

Type variable can be used in any expression, including assignment, function signature, struct field declaration, etc.

```go
// Use type variable and value variable in one expression.
T, i := int, 0

// Use type variable in function signature.
func F(T generic, i int) (S generic, ok bool) {}

// Use type variable in struct.
type S struct {
	T     generic
	Value T       // Use T in struct context.
}

// Use type variable in type assertion.
T := int
var i interface{} = 1
i.(T) // Get an int value.
```

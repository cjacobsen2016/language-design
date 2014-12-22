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

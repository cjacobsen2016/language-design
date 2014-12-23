## Modify type variable ##

Type variable can be modified freely in current code. If it's referred in a child block, it's mutable only in following blocks.

* A solely block;
* A branch of a `if` statement in which condition can be evaluated at compile-time.

```go
const N = 1

func K(i int) generic {
	T := int

	// This is a solely block.
	{
		// Valid as the block can be inlined.
		T = int8
	}

	if T == float64 {
		// Valid. `T == float64` is a compile-time constant.
		return float32
	}

	if N == 1 {
		// Valid. `N == 1` is a compile-time constant.
		return uint8
	}

	if i == 0 {
		// Compile-time error.
		// `i == 0` cannot be evaluated at compile-time.
		T = int16
	}

	if i == 0 {
		// Compile-time error.
		// Return value is implicitly modified.
		return T
	}

	return float64
}
```

When a type variable is immutable in a block, type variable value is determined at the first time it's referred.

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
	// Compile-time error.
	// T is defined in function signature. It's immutable.
	T = int
}

func N(i int) {
	T := int

	if i == 0 {
		T = float64 // Compile-time error. T is locked and immutable.
	}
}
```

It's always meanless to declare a zero value type variable in global scope. So it's a Compile-time error if one tries to do this.

```go
// Declare a zero value type variable in global scope is an error.
var T generic
```

## Instantiate generic struct ##

Generic struct must be instantiated explicitly or implicitly before constructing a value.

```go
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

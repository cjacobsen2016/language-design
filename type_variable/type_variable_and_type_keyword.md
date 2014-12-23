## Type variable and `type` keyword ##

Type declaration using `type` keyword works in Rat. The type literal defined by `type` is immutable.

Type declaration always creates a new and different type variable from underlying type as the type identity is different. This is not new in Rat. It's [well defined](http://golang.org/ref/spec#Type_identity) in Go.

Assignment is different. It copies type variable "reference" from one to another, so type identity will not change after copy.

```go
// T and int are different.
type T int

T1 := int // T1 is int.
T2 := T   // T2 is T.

T  == int // false
T1 == T2  // false
T1 == int // true
T2 == T   // true

T = float32 // Compile-time error. T is immutable.
```

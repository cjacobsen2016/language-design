## Operators ##

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

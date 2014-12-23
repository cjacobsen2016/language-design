## Type variable and `type` keyword ##

Type variable can store any valid type. On the other hand, any type, including predeclared types, can be treated as a type variable. Any valid Go type is a type variable and can be used as a variable.

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

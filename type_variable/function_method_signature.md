## Function/method signature ##

Type variable can be used as type in function or method signature, both parameters and result.

```go
func F(T generic) (R generic) {
	return bool
}

func (s *S) M(T generic) {}

R := F(int) // Provide type variable as parameter.
R == bool   // true

s := &S{}
s.M(bool)
```

A defined type variable can be used as type in signature.

```go
func F(T generic, t T) {} // Valid.
func F(T generic) T {}    // Valid.
func G(t T, T generic) {} // Compile-time error.
                          // T must be defined before using it.
```

Use blank identifier `_` to indicate that a type variable can be deduced by other parameters. If compiler fails to deduce the type, it will raise a compile-time error.

```go
func F(T generic, t1, t2 T) {}
func G(T generic) G {}

F(_, 1, 2)   // Valid. as type of t1 and t2 is int, T is int.
F(_, 1, 2.0) // Compile-time error.
             // Type of t1 is int but Type of t2 is float64. Type conflict.

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
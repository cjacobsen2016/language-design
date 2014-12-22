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

Type of `yield` declared in result must be a function. This function cannot take any type variable in function signature, both parameters and result.

```go
// Compile-time error. Type of `yield` cannot take any type variable.
func F() (yield func(T generic)) {}
```

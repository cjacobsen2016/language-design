## Type variable field ##

There is no special syntax for type variable field declaration. It's just a field whose type is `generic`.

Type variable field must have a unique and non-blank name within a struct. Type variable field cannot be anonymous.

```go
type S struct {
	T1, T2 generic             // Valid.
	T3     generic `foo:"bar"` // Valid.
	_      generic             // Compile-time error. Name cannot be blank identifier.
	T1     int                 // Compile-time error. Duplicated name `T1`.

	generic // Compile-time error. Generic itself is not a type.
	T3      // Compile-time error. Conflict name with `T3 generic`.
}
```

Type variable field can be read through a constructed struct value.

```go
type S struct {
	T generic
}

v1 := S{T: int}
v2 := &S{T: float32}
v3 := &v2

T1 := v1.T    // T1 is int.
T2 := v2.T    // T2 is float32.
T3 := (*v3).T // T3 is float32.
```

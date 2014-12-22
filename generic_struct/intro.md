## Generic struct ##

Generic struct is a struct with one or more type variables in field declaration.

```go
// As S has a type variable T as a field, it's a generic struct.
type S struct {
	T generic
	V T
}
```

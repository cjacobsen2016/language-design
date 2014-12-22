## Built-in function `new` ##

A type must implement `NewOverrider` to override `new`. Method receiver is initialized to zero value of receiver type.

```go
type NewOverrider interface {
	generic() Receiver
	OverrideNew() Receiver
}
```

Following is an example.

```go
type S struct {
	V int
}

// Override `new`.
func (s *S) OverrideNew() *S {
	// Receiver `s` is set to zero value.

	// Set s.V to a special value.
	return &S{
		V: 1,
	}
}

s := new(S)
s.V == 1 // true
```

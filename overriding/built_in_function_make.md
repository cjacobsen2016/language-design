## Built-in function `make` ##

A type must implement `MakeOverrider` to override `make`. Method receiver is initialized to zero value of receiver type.

```go
type MakeOverrider interface {
	generic(...Args) Receiver
	OverrideMake(Args) Receiver
}
```

Following is an example.

```go
type S struct {
	Data []int
}

// Override `make`.
func (s *S) OverrideMake(size, cap int) *S {
	// Receiver `s` is set to zero value.

	// Set s.V to a special value.
	return &S{
		Data: make(generic(S.Data), size, cap),
	}
}

s := make(S, 10, 20)
len(s.Data) == 10 // true
cap(s.Data) == 20 // true
```
## Built-in function `len` ##

A type must implement `LenOverrider` to override `len`. Method receiver is set to the first parameter of `len`.

```go
type LenOverrider interface {
	OverrideLen() int
}
```

Following is an example.

```go
type T []int

// Override `len`.
func (t T) OverrideLen() int {
	return 10
}

var t T
len(t) == 10 // true
```

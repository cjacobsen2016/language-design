## Built-in function `cap` ##

A type must implement `CapOverrider` to override `cap`. Method receiver is set to the first parameter of `cap`.

```go
type CapOverrider interface {
	OverrideCap() int
}
```

Following is an example.

```go
type T []int

// Override `cap`.
func (t T) OverrideCap() int {
	return 10
}

var t T
cap(t) == 10 // true
```
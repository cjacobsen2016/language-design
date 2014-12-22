## Built-in function `close` ##

A type must implement `CloseOverrider` to override `close`. Method receiver is set to the first parameter of `close`.

```go
type CapOverrider interface {
	OverrideClose()
}
```

Following is an example.

```go
type T chan int

// Override `close`.
func (t T) OverrideClose() int {
	println("Closing...")

	// Cast T to a plain chan type to avoid recursive call.
	var c chan int = t
	close(c)
}

t := make(T)
close(t) // Output: Closing...
```

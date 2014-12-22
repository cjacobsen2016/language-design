## Send statement ##

A type must implement `SendOverrider` to override send statement. Method receiver is set to the channel expression.

```go
type SendOverrider interface {
	generic(Value)
	OverrideSend(Value)
}
```

Following is an example.

```go
type T int

func (t T) OverrideSend(value int) {
	t += T(value)
}

var t T

t <- 10
t <- 20

println(t) // Prints: 30
```

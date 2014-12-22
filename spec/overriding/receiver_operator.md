## Receiver operator ##

A type must implement `ReceiverOverrider` to override receiver operator. Method receiver is set to the value of operand. If receiver operator is called in a double-value context, `OverrideReceiverOK` will be called. Otherwise, `OverrideReceiver` is called.

```go
type ReceiverOverrider interface {
	generic(Value)
	OverrideReceiver() Value
	OverrideReceiverOK() (Value, bool)
}
```

Following is an example.

```go
type T int

func (t T) OverrideReceiver() int {
	i, ok := t.OverrideReceiverOK()

	if !ok {
		panic("Exhausted.")
	}

	return i
}

func (t T) OverrideReceiverOK() (int, bool) {
	if t <= 0 {
		return 0, false
	}

	t--
	return t, true
}

t := T(3)

println(<-t) // Prints: 2
println(<-t) // Prints: 1
println(<-t) // Prints: 0
println(<-t) // Panic.
```

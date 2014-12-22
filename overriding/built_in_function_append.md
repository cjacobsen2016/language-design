## Built-in function `append` ##

A type must implement `AppendOverrider` to override `append`. Method receiver is set to the first parameter of `append`.

```go
type AppendOverrider interface {
	generic(Arg) Receiver
	OverrideAppend(...Arg) Receiver
}
```

Following is an example.

```go
type S struct {
	Data []int
}

// Override `append`.
func (s *S) OverrideAppend(data ...int) *S {
	s.Data = append(s.Data, data...)
	return s
}

s := &S{}
s = append(s, 1, 2)
println(s.Data) // Prints: [1 2]
```

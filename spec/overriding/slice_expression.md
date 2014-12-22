## Slice expression ##

A type must implement `SliceOverrider` to work with slice expression. Method receiver is set to the sliced value. When low is not set in expression, it will be 0 in method parameter. When high is not set in expression, it will be `len(value)` in method parameter.

```go
type SliceOverrider interface {
	generic() Receiver
	OverrideSlice(low, high int) Receiver
	OverrideSlice3(low, high, max int) Receiver
}
```

Following is an example.

```go
type S struct {
	Data []int
}

func (s *S) OverrideSlice(i, j int) *S {
	return &S{
		Data: s.Data[i:j],
	}
}

func (s *S) OverrideSlice(i, j, k int) *S {
	return &S{
		Data: s.Data[i:j:k],
	}
}

s := &S{}
s.Data = []int{1, 2, 3, 4, 5}
s1 := s[1:3]
println(s1.Data)      // Prints: [2 3]
s2 := s[1:3:4]
println(cap(s2.Data)) // Prints: 3
```

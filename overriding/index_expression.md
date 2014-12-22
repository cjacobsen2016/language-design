## Index expression ##

A type must implement `IndexOverrider` to work with slice expression. Method receiver is set to the indexed value. When index expression is called in a double-value context, `OverrideGetItemOK` will be called. Otherwise, `OverrideGetItem` is called for expression and `OverrideSetItem` is called for assignment.

```go
type IndexOverride interface {
	generic(Index, Value)

	OverrideGetItem(Index) Value
	OverrideGetItemOK(Index) (Value, bool)
	OverrideSetItem(Index, Value)
}
```

Following is an example.

```go
type S struct {
	Data map[string]int
}

func (s *S) OverrideGetItem(index string) int {
	return s.Data[index] + 2
}

func (s *S) OverrideGetItemOK(index string) (int, bool) {
	v, ok := s.Data[index]

	if !ok {
		return 0, false
	}

	return v + 2, true
}

func (s *S) OverrideSetItem(index string, value int) {
	s.Data[index] = value + 2
}

s := &S{}
s["foo"] = 2
println(s["foo"]) // Prints: 6
_, ok := s["bar"]
println(ok)       // Prints: false
```

## Built-in function `delete` ##

A type must implement `DeleteOverrider` to override `delete`. Method receiver is set to the first parameter of `delete`.

```go
type DeleteOverrider interface {
	generic(Index)
	OverrideDelete(Index)
}
```

Following is an example.

```go
type S struct {
	M map[string]int
}

// Override `delete`.
func (s *S) OverrideDelete(index string) {
	delete(s.M, index)
}

s := &S{}
s.M = make(generic(s.M)) // Make a map value.
s.M["foo"] = 123

delete(s, "foo")
s.M["foo"] == nil // true
```

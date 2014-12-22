## Range expression ##

A type must implement `RangeOverrider` to work with `range` expression. Method receiver is set to the expression value in `range` clause.

```go
type RangeOverrider interface {
	generic(...Args)
	OverrideRange() (yield func(Args))
}
```

Following is an example.

```go
type S struct {
	Data []int
}

// Override `range` expression.
func (s *S) OverrideRange() (yield func(int, int)) {
	for i := range s.Data {
		yield(i, i * 2)
	}
}

s := &S{}
s.Data = []int{1, 2, 3, 4}

// Output:
// 1	2
// 2	4
// 3	6
// 4	8
for i, j := range s {
	fmt.Println(i, j)
}
```

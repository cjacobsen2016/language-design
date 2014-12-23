## Method declaration ##

Method declaration syntax is the same as that in Go. Type of method parameters can be a type variable field.

```go
type Array struct {
	T generic
	data []T
}

// arr.T can be used as parameter type.
func (arr *Array) Append(values ...arr.T) {
	arr.data = append(arr.data, values...)
}

// arr.T can be used as return type.
func (arr *Array) Get(index int) arr.T {
	return arr.data[index]
}
```

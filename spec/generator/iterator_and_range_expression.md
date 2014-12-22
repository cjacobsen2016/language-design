## Iterator and `range` expression ##

Iterator value returned by generator can work with `range` expression. Type of iteration variables in `range` clause are determined by `yield` parameter types. Count of iterator variables must be not greater than that of `yield` parameters.

```go
func Gen() (yield func(int, float64)) {
	yield(1, 2.3)
	yield(2, 3.4)
}

func main() {
	// Output:
	// 1	2.3
	// 2	3.4
	for i, j := range Gen() {
		fmt.Println(i, j)
	}

	// Output:
	// 1
	// 2
	for i := range Gen() {
		fmt.Println(i)
	}

	// Output:
	// 2.3
	// 3.4
	for _, j := range Gen() {
		fmt.Println(j)
	}
}

func illegal() {
	// Compile-time error. Gen's `yield` only yields two values at most.
	for i, j, k := range Gen() {
		// ...
	}
}
```

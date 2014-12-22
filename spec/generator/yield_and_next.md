## `yield` and `Next` ##

When caller calls a generator, the generator will return an iterator value. The returned iterator has `Next` method. Method signature is determined by `yield` type in generator. Specifically, `Next` method's result type is all `yield` parameters plus a bool indicating done; `Next` method parameters is `yield` result.

```go
func Gen() (yield func(int, float32) string) {}

iter := Gen()

// Next signature is func(string) (int, float32, bool).
iter.Next("foo")
```

Caller uses iterator value's `Next` method to get value from generator. On the other hand, generator uses `yield` as a function inside function body to generate value one by one. Once `yield` is called, generator will be halted until caller consumes the value. If generator returns using `return` statement or is panic, future calls to `Next` will always return zero yielded values plus a true, which means iteration is done.

```go
func Gen() (yield func(int)) {
	fmt.Println("Gen: before yield.")
	yield(1)
	fmt.Println("Gen: 1st yield is called.")
	yield(3)
	fmt.Println("Gen: 2nd yield is called.")
}

func main() {
	iter := Gen()

	// As `yield` type is func(int), type of Next method is func() (int, bool)
	fmt.Println("main: before first Next.")
	fmt.Println(iter.Next())
	fmt.Println("main: 1st Next is called.")
	fmt.Println(iter.Next())
	fmt.Println("main: 2nd Next is called.")
	fmt.Println(iter.Next())
	fmt.Println("main: 3rd Next is called.")
	// Output:
	// main: before first Next.
	// Gen: before yield.
	// 1	false
	// main: 1st Next is called.
	// Gen: 1st yield is called.
	// 3	false
	// main: 2nd Next is called.
	// Gen: 2nd yield is called.
	// 0	true
	// main: 3rd Next is called.
}
```

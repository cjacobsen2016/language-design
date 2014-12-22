## Exchange values ##

When `yield` is a function returning values, caller can use `Next` method to set values returned by `yield`.

First call to `Next` method is special. A generator can only get returning values by calling `yield`. As code in generator doesn't start executing until `Next` is called for first time, there is no `yield` at begining thus no way to retrieve first `Next` parameters in generator. It's a by-design behavior.

```go
func Gen() (yield func() int) {
	fmt.Println("yield:", yield())
	fmt.Println("yield:", yield())
}

func main() {
	iter := Gen()
	iter.Next(1) // Parameters in first Next cannot be retrieved in generator.
	iter.Next(2)
	iter.Next(3)
	// Output:
	// yield: 2
	// yield: 3
}
```

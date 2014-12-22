## Iterator interface ##

All iterator values implement `Iterator` type. This type is defined in `github.com/go-rat/rat/types`.

```go
// Defined in "github.com/go-rat/rat/types".
type Iterator interface {
	generic(...Yield, ...Result)
	Next(Result) (Yield, bool)
}

func Gen() (yield func(int)) {}

// Valid.
var it Iterator = Gen()
it.Next()
```

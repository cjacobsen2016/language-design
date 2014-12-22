## Overriding ##

Many `go` built-in functions and operators only work with built-in types like map, slice and chan. `Rat` extends them to work with custom types by overriding mechanism.

Override mechanism is based on interface types. Every built-function or operator, which can be overridden, has one special interface for overriding. If a type implements the interface, it can override the respective function or operator.

If one decides to use overriding feature, one should make sure semantics of overridden function or operator is not changed.

For example, built-in function `cap` expects a value implements `CapOverrider` interface, which is defined as following.

```go
type CapOverrider interface {
	OverrideCap() int
}
```

If a type implements `CapOverrider`, `CapOverrider` will be called when `cap` is called against type's value.

```go
type Array []int

func (arr Array) OverrideCap() int {
	return 10
}

var a Array
println(cap(a)) // Always prints 10.
```

All interfaces related to overriding are defined in `github.com/go-rat/rat/types`.
### Embedding generic interface ###

If a generic interface is embedded in another interface, all type variables defined in this interface must also be defined in the interface embedding it. So, only generic interface can embed generic interface.

```go
type I1 interface {
	generic(T)
	M(T)
}

// Valid.
type I2 interface {
	generic(T)
	I1
}

// Compile-time error. I3 doesn't have a type variable named T.
type I3 interface {
	generic(S)
	I1
}
```

## Variadic type variables ##

The `generic` method in a generic interface can have any number of variadic type variables. Each of them can represent any number of parameters in a method. Once a generic interface is instantiated, variadic type variables are set to a specific list of parameters.

```go
type I interface {
	generic(...Args)

	// M1 and M2 must have the same parameters in type.
	M1(Args)
	M2(Args)
}

// T1 implements I.
type T1 int
func (t1 T1) M1(int, bool) {}
func (t1 T1) M2(int, bool) {}

// T2 doesn't implement I as parameters of M1 and M2 are different.
type T2 int
func (t2 T2) M1(int, bool) {}
func (t2 T2) M2(float64, string) {}
```

Mind that variadic type variable is not a type variable. It's not valid to use "..." in method with variadic type variable.

```go
type I interface {
	generic(...Args)

	// Compile-time error.
	// Args is not a type variable. Adding "..." doesn't make any sense.
	InvalidM(...Args)
}
```

Variadic type variables can appear in any position of parameters or result. Compiler will try its best to find a matchable pattern.

```go
type I interface {
	generic(...Args, ...Return)
	M1(int, Args) (Return, error)
}

// T implements I.
// Args is instantiated as (float64, string).
// Result is instantiated as (T, bool).
type T int
func (t T) M1(int, float64, string) (T, bool, error) {}
```

Two variadic type variables in generic interface cannot be used in one parameters or result declaration, as there is no way for compiler to decide how to instantiate these type variables.

```go
type I interface {
	generic(...Args1, ...Args2)

	// Compile-time error. Args1 and Args2 cannot be used in one parameters declaration.
	InvalidM(Args1, Args2)
}
```

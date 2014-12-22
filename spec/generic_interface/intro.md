## Generic interface ##

Generic interface is an interface contains a method with name `generic`. All types, not variables, in this `generic` method are type variables. They can be used in all methods of this interface.

```go
// A generic interface.
type I interface {
	generic(T)
	F(T)
}
```

Generic interface can be used to check whether a generic struct implements expected methods. It cannot be used in any type assertion as it requires runtime information.

```go
type I interface {
	generic(T)
	F(T)
}

// S1 implements I.
type S1 struct {}
func (s1 S1) F(v int) {}

// S2 also implements I.
type S2 struct {
	T2 generic
}
func (s2 S2) F(v s2.T2) {}

// S3 doesn't implement I, as S3#F takes 2 parameters instead of 1.
type S3 struct {
	T3 generic
}
func (s3 S3) F(T generic, v int) {}
```

When a generic interface is fully instantiated, it works exactly the same an non-generic interface.

```go
type I interface {
	generic(T)
	Add(T)
}

IInt := generic(I, int)

// S implements IInt.
type S struct {}
func (s S) Add(int) {}

var s S
s.(IInt) // OK.
```

If `generic` method has result, the result will be instantiated as method receiver's type. As there is only one receiver for a method, it's meanless to declare more than one type variables in result.

```go
type I interface {
	generic() Receiver
	New() Receiver
}

// T1 implements I.
type T1 int
func (t1 T1) New() T1

// T2 doesn't implement I.
// Return type doesn't match receiver type as T2 != int.
type T2 int
func (t2 T2) New() int {}
```

Type variable cannot be used in interface method other than `generic`.

```go
type I interface {
	InvalidM(T generic) // Compile-time error.
}
```

## Overview ##

Go does have generic.

Look at `map` and `chan`. Both of them can be "instantiated" with any provided type. Unfortunately, such generic syntax doesn't work with custom types. Type `map[int]string` or `chan int` is valid but `MyStruct[Foo]Bar` or `MyStruct Foo` is not. Go compiler treats `map` and `chan` as keywords rather than type identifier. Moreover, such syntax is not extensible. It's nearly impossible to extend it to support any number of generic types in one type.

It's' easy to borrow generic/template syntax from other languages, but it may not work well with Go. For example, interfaces may be broken due to method overloading caused by generic type instantiation.

Is it possible to find out a Go-ish design for generic?

The answer is positive. That's the "type variable" introduced by Rat.

Thinking of the essence of generic: Reuse code with different types in different context. This is a specific problem only in static typed language, as type must be hard-coded in code. If a type can be used as a variable, the problem will be resolved automatically.

In Rat, every type is a varaible. The type of type variable is `generic`, a special type introduced by Rat. Type variable can be used in any expression like value variable. Function can take type variable as parameter or return type variable as result. Struct can have type variable as field.

When implementing a generic slice, one can write following code.

```go
type Slice struct {
	ElemType generic    // ElemType is a type variable.
	Data     []ElemType // Data is a slice of ElemType.
}

// Use type variable `s.ElemType` as a field and a type.
func (s *Slice) Append(data ...s.ElemType) *Slice {
	s.Data = append(s.Data, data...)
	return s
}

// Instantiate Slice's ElemType with int.
s := &Slice{
	ElemType: int,
}

s = s.Append(1, 2, 3) // Append's type is `func(...int) *Slice`.
println(s.Data)       // Prints: [1 2 3]
```

There is no new syntax for type variable. Rat just introduces a new type to represent type itself. With this design, Rat can keep 100% compatibility with Go at syntax level. Gophers should feel quite familiar with Rat at the first glance. Packages like `go/scanner` can work with Rat source files without any modification.

Rat also introduces generator and overriding. With all these features, one is able to implement data structure and use it in the same way as built-in `map`, `chan`, array or slice.

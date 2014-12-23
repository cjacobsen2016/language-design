## Generic struct as field ##

When type of a struct field is a generic struct, field type can use type variables defined in the struct by name to get instantiated implicitly. If field type cannot find a type variable with expected name, field type cannot be instantiated thus cannot be constructed.

```go
type List struct {
	ValueType  generic
	Head ListNode      // ListNode use the ValueType to instantiate itself.
}

type ListNode struct {
	ValueType  generic
	Prev, Next *ListNode
	Value      ValueType
}

list := &List{
	ValueType: int,
}
list.ValueType      == int // true
list.Head.ValueType == int // true

type WiredList struct {
	T    generic
	Head ListNode // Compile-time error. Cannot use ListNode without instantiation.
}

type SuperListNode struct {
	ValueType generic
	extra     interface{}
	ListNode              // Generic struct can be an anonymous field.
}
```

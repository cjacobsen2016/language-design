# Design Principles #

The `rat` is a new language based on go. It treats go as a kind of asm. All `rat` files can be compiled into go source files.

The main design goal of `rat` is to allow developers to implement generic containers.

Writing container library in go is quite painful without generic. One has to use `interface{}` to store all data types, or write lots of similar structs/functions for all necessary types. `Rat` will define and implement a compile time generic like C++ template. One can write code once for all types.

The main goal is the only goal. All others are non-goals. Adding new features to a newly designed language is quite easy. Let us make it hard at begining. `Rat` should always be lean and focused.

To be more specific, if a feature is planned in `rat`, it must be a reasonable necessary part to implement following generic `Map`.

(Note: Syntax in the demo may change in future.)

```go
type Map struct { /* Suppose we have defined a generic Map. */ }

// Make a new map and provide some arguments for construction.
// Can be instantiated as an expression.
// Handle built-in function `make` and add some logic after made.
m := make(Map(string, int), 10)

// Set item by operator `[]`.
m["foo"] = 12

// Get item by operator `[]`.
_ := m["foo"]

// Return two values when getting an item.
_, ok := m["foo"]

// Handler built-in function `delete`.
delete(m, "foo")

// Iterate over Map elements.
for k, v := range m {
	// ...
}

// Provide struct function just like a normal struct.
m.Clear()
```

## Design Principles ##

Major design goal of Rat is to allow developers to implement generic containers. Writing container library in Go is quite painful. One has to use type-unsafe `interface{}` to store all data, or write lots of similar structs/functions for all necessary types. Rat will address this issue to let developers write less do more.

The major goal is the only goal. All others are non-goals. Adding features to a newly designed language is quite easy. Let me make it hard at very begining. Rat should always be very lean and focused.

To be more specific, if a feature is planned in Rat, it must be a necessary part to implement following generic `Map` which works the same as built-in `map`. If not, the feature will be rejected.

(Note: Syntax in the demo may change in future.)

```go
type Map struct { /* Suppose we have defined a generic Map. */ }

// Make a new map and provide some arguments for construction.
// Can be instantiated as an expression.
// Handle built-in function `make` and add some logic after made.
m := make(generic(Map, string, int), 10)

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

// Generic struct can have methods.
m.Clear()
```
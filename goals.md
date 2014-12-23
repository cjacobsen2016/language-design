## Design Goals ##

Major design goal of Rat is to enable developers to implement generic data structures.

Writing a data structure library is quite painful in Go. One has to use type-unsafe `interface{}` to store all data, or write lots of similar structs/functions for all necessary types. Rat is designed to address this issue.

The major goal is the only goal. All others are non-goals. Adding features to a new language is quite easy. Let me make it hard at very begining. Rat should always be very lean and focused.

To be more specific, if a feature is planned in Rat, it must be a necessary part to implement following generic `Map` which works in the same way as built-in `map`. If not, the feature will be rejected.

(Note: Syntax in the demo may change in future.)

```go
// Suppose we have a generic Map struct.
type Map struct { /* ... */}

// Make a new Map and provide some arguments for construction.
m := make(generic(Map, string, int), 10)

// Set item through operator `[]`.
m["foo"] = 12

// Get item through operator `[]`.
_ := m["foo"]

// Get item in two-value context.
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

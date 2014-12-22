## Motivation ##

There are a lot of threads talking about generics in `go` on gopher-nuts mailing list. Response from go authors is always that go won't have generics.

Personally, I don't want to add generic to `go` either. This feature will significantly slow down `go` build speed. Tracking and instantiate generic types is quite expensive. As long as build speed is a goal, `go` will not have generic or similar features.

However, nothing can stop me to create a new language using `go` syntax with generic support. It will be useful for people who decide to sacrifice build time for writing less code. So this is why I start to write `rat`.

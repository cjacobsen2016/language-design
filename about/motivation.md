## Motivation ##

There are a lot of [threads](https://groups.google.com/forum/#!searchin/golang-nuts/generic/golang-nuts/y3LqthbBiuY/tJZw232o7ggJ) [talking](https://groups.google.com/forum/#!searchin/golang-nuts/generic/golang-nuts/smT_0BhHfBs/jhc9BdQc5w0J) [about](https://groups.google.com/forum/#!searchin/golang-nuts/generic/golang-nuts/7xeflQp_IdA/VC-OvVw1_wEJ) [generic](https://groups.google.com/forum/#!searchin/golang-nuts/generic/golang-nuts/L-2OTItScJk/LAy6-EJe2UIJ) on golang-nuts mailing list. Response from go authors is always that go won't have generic.

Personally, I don't want to add generic to Go either. This feature will significantly slow down Go build speed. Tracking and instantiating generic types is quite expensive. As long as build speed is a goal, Go should not have generic or similar features.

However, nothing can stop me to create a new language using Go syntax with generic support. It will be useful for people who decide to sacrifice build time for writing less code. So this is why I start to write Rat.

## Do not abstract by visual pattern matching
Two pieces of code that are identical but represent two distinct concepts should stay separated: they are not duplication.
Let's consider the example where we are asked to calculate the perimeter of rectangles and rhombuses.

```kotlin
class Rhombus(private val width: Int, private val height: Int) {

  fun perimeter(): Int {
    return 2 * (width + height)
  }
}

class Rectangle(private val width: Int, private val height: Int) {

  fun perimeter(): Int {
    return 2 * (width + height)
  }
}
```

When seeing a piece of code like the above, one might be tempted to remove the duplication of the `perimeter` method like follows.

```kotlin
class Shape(private val width: Int, private val height: Int) {

  fun perimeter(): Int {
    return 2 * (width + height)
  }
}
```

However, what happens if after some time we are asked to calculate the perimeter of a circle? Now the concept represented
by `Shape` does not relate to circle, which is definitely a shape though. There are many ways to refactor this situation,
for instance making shape an interface, but in a large codebase these refactorings are more expensive than leaving duplicated code
until we understand it is duplicated knowledge. It is hard to build the right knowledge model, especially if specifications
are not entirely known or the domain is complex. In doubt, the rule of thumb is to remove duplication only after it occurs
more than two times.

[1] ["duplication is far cheaper than the wrong abstraction", RailsConf 2014, all the little things talk - Sandy Metz](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction)  
[1] [Rule of three, When we should Refactor? chapter 2 of Refactoring - Martin Fowler](https://www.goodreads.com/en/book/show/44936.Refactoring)  
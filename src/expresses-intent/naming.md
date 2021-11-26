## Naming
Great naming let us understand what the code means without inspecting its details. In fact, we should be able
to understand an entire codebase just reading the names of classes, methods and variables, never looking at what they contain.
This clarity is reached by naming classes, methods and variables after what they do instead of how they do it.
The how can be looked up later if more details are needed by inspecting the content. The benefits are two:
* names do not contain any noise
* if the implementation changes, the name remains relevant

Let's consider the following class

```kotlin
class BiDimensionalCoordinates(private val x: Int, private val y: Int) {
  
  fun sumByXandY(addend: BiDimensionalCoordinates): BiDimensionalCoordinates {
    return BiDimensionalCoordinates(x + addend.x, y + addend.y)
  }
}
```

In the above code, both class and method names leak the how. Given the distinction we made about the what and the how,
we can ask ourselves two questions. The first one is: if we remove the how from the `BiDimensionalCoordinates` names, does the class
still express intent? Let's see.

```kotlin
class Coordinates(private val x: Int, private val y: Int) {
  
  fun sum(addend: Coordinates): Coordinates {
    return Coordinates(x + addend.x, y + addend.y)
  }
}
```

The answer is yes, as the above class still expresses clear intent, without even bothering the reader with the noise of the how.
The second questions is: would the naming in `BiDimensionalCoordinates` still make sense if we were to switch to three-dimensional
coordinates? Let's see.

```kotlin
class BiDimensionalCoordinates(private val x: Int, 
                               private val y: Int, 
                               private val z: Int) {
  
  fun sumByXandY(addend: BiDimensionalCoordinates): BiDimensionalCoordinates {
    return BiDimensionalCoordinates(x + addend.x, y + addend.y, z + addend.z)
  }
}
```

The answer is no. `BiDimensionalCoordinates` now does not express its intent anymore as it is lying to the reader:
the names suggest two dimensions coordinates when they are actually three. However, the `Coordinates` class would still express
its intent even when using three dimensions as shown below.

```kotlin
class Coordinates(private val x: Int,
                  private val y: Int,
                  private val z: Int) {
  
  fun sum(addend: Coordinates): Coordinates {
    return Coordinates(x + addend.x, y + addend.y, z + addend.z)
  }
}
```

As a final note for typed languages, be mindful that the return type of a method is part of its naming. A method `sum`
does not clearly expresses intent if its signature is like the following: `fun sum(addend: Coordinates): Int`

<br/>  

#### Recommended reads
* [Names, section 1.1.1 of 99 bottles of OOP - Sandy Metz](https://www.goodreads.com/book/show/31183020-99-bottles-of-oop)  
* [Choosing Names, section 2.8 of 99 bottles of OOP - Sandy Metz](https://www.goodreads.com/book/show/31183020-99-bottles-of-oop)  
* [Meaningful Names, chapter 2 of Clean Code - Robert C. Martin](https://www.goodreads.com/book/show/3735293-clean-code)  
* [Replace Magic Number with Symbolic Constant, chapter 8 of Refactoring - Martin Fowler](https://www.goodreads.com/en/book/show/44936.Refactoring)  
* [Developing a Ubiquitous Language, chapter 2 of Domain-Driven Design Distilled - Vaughn Vernon](https://www.barnesandnoble.com/w/domain-driven-design-distilled-vaughn-vernon/1124175630)

#### Teach me back
I really appreciate any [feedback](../introduction/introduction.html#teach-me-back) about the book and my current understanding of software design.
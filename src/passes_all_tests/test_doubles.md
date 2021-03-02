## Test doubles
When testing a class, often we need to create a lot of objects just for the sake of the test. This happens for two reasons:
* All those objects are needed to make the test run (e.g. they are required by the constructor of the class under test)
* We want to verify how the class under test interacts with those objects (e.g. if the class under test calls the public method of another one)

Because of the above, tests can become time-consuming and tedious, but we can remediate using test doubles. For the sake
of development speed, test doubles are usually created via testing libraries. For instance, in Kotlin there is
[MockK](https://mockk.io/) among others. Taking the following code as example, let's define the different types of test doubles
and how they look like in a test when using a library like MockK.

```kotlin
class Greeter(private val validator: Validator) {

  fun greetings(): String {
    return "Hello"
  }

  fun personalisedGreetings(name: String): String {
    if (validator.isValid(name))
      return "Hello $name"
    
    return "Not a valid name"
  }
}

class Validator {

  fun isValid(name: String): Boolean {
    if (name.isEmpty())
      return false
    
    return true
  }
}
```

**Stubs**  
They return a hardcoded response. In the following test, `validator` is a stub. Unfortunately mocks
and stubs in MockK are both defined as `mockk<>()` which makes it confusing for newcomers.

```kotlin
@Test
  fun `greets by name`() {
    val validator = mockk<Validator>()
    every{ validator.isValid("Andrea") } returns true
    val controller = Greeter(validator)
  
    val result = controller.personalisedGreetings("Andrea")
  
    assertEquals("Hello Andrea", result)
  }
```  

**Mocks**  
They have two responsibilities:
* they return a hardcoded response
* the test checks that their public methods are called with specific input parameters

In the following test,
`validator` is a mock.

```kotlin
@Test
  fun `performs successful validation on the name`() {
    val validator = mockk<Validator>()
    every{ validator.isValid("Andrea") } returns true
    val controller = Greeter(validator)
  
    controller.personalisedGreetings("Andrea")
  
    verify{ validator.isValid("Andrea") }
  }
```

**Spies**  
The test checks that their public methods are called with specific input parameters. In the
following test, `validator` is spy.

```kotlin
@Test
  fun `performs validation on the name`() {
    val name = "Andrea"
    val validator = spyk<Validator>()
    val controller = Greeter(validator)

    controller.personalisedGreetings(name)

    verify{ validator.isValid(name) }
  }
```
**Dummies**  
They are used to run the test but they do not take any part in it, meaning no public method of theirs is called.
For instance, `validator` in the following test is a dummy as the method `greetings` does not interact with `validator`.

```kotlin
@Test
  fun `greets by saying Hello`() {
    val validator = mockk<Validator>()
    val controller = Greeter(validator)

    val result = controller.greetings()

    assertEquals("Hello", result)
  }
```
**Fakes**  
An object with very limited capabilities compared to the real one but much faster to create. The typical example is
an in memory database (e.g. H2) instead of a production one (e.g. PostgreSQL)


As a final note, test doubles are not used just for unit tests, but throughout the [whole pyramid](#test-pyramid).

[1] [Mocks aren't stubs](https://martinfowler.com/articles/mocksArentStubs.html)  
[2] [Only mock type that you own, chapter 8 of Growing Object-Oriented Software, Guided by Tests - Steve Freeman, Nat Pryce](https://www.goodreads.com/book/show/4268826-growing-object-oriented-software-guided-by-tests)]


### Test driven development
Test driven development (TDD) is an approach to development where tests are written before the actual code. The core of TDD is a three
steps process:
1. Write a failing test
2. Make the test pass
3. Refactor

Let's go through each step using the following specifications as example:
* I want measure things in meters
* I want to be able to some two measurements in meters

**Write a failing test**  
At this step we pick a single aspect of the specification we want to verify: the smallest the better. The common mistake
here is to overthink it because we want to take into account all the specifications and their nuances. Don't do it now,
the third step will take care of it. For now let's just write a test even if it seems silly for how simple it is. Moreover,
don't focus on writing good code either: the third step will take care of this as well. Finally, remember to run the test
and see it failing with the error we expect: we do not want to later discover that our code is passing tests by luck.

```kotlin
@Test
fun `summing 1 meter and 2 meters returns 3 meters`() {
  val result = MeasureInMeters(1).add(MeasureInMeters(2))

  assertEquals(MeasureInMeters(3), result)
}
```


**Make the test pass**  
Now that we have a failing test we need to make it pass. Be mindful that compilation errors in the test is the same of
running it and see it fail. At this step as at the previous, let's not focus about writing good code, that's something for
the third step: just make the test pass, whatever it takes.


```kotlin
data class MeasureInMeters(private val amount: Int) {

  fun add(measureInMeters: MeasureInMeters): MeasureInMeters {
    return MeasureInMeters(3)
  }
}
```

(In Kotlin the `data class` keyword makes sure that two instances of `Meter` are equal if their `amount` is equal).


**Refactor**  
In this step we improve the test and the actual code we have written. Let's start from the naming.

```kotlin
@Test
fun `adding 1 meter and 2 meters returns 3 meters`() {
  val result = Measure(1).add(Measure(2))

  assertEquals(Measure(3), result)
}

data class Measure(private val amount: Int) {

  fun add(measure: Measure): Measure {
    return Measure(3)
  }
}
```

Let's also generalise the body of the method `sum` for all integers as we already understand it won't work for another
pair of numbers.

```kotlin
data class Measure(private val amount: Int) {

  fun add(measure: Measure): Measure {
    return Measure(amount + measure.amount)
  }
}
```

The third step is also when look at the bigger picture. In this example there is little to do, but usually this is the moment
when we pay attention on how the new code we are writing fits into the existing codebase (e.g. knowledge
duplication, edge cases for some other classes we did not think before, etc.). We do not need to address these concerns
right away, but it is good to note them down so we can tackle them once we have done satisfying the bit of specifications
we are currently working on. Once we are happy with the quality of the code we have just written, we can restart
the three steps process. The focus will be either another test for the bit of specification we are focusing on
(e.g. adding 2 negative integers) or a test for another bit of the specifications.

Compared to writing tests after the implementation, the main benefit of TDD is that it becomes much harder to write code
that is not tested. However, be mindful that we can still write buggy code even if we use TDD as described in the
[test coverage is not enough](#test-coverage-is-not-enough) section.

<br/>  

#### Recommended reads
* [Test-Driven Development: By Example - Kent Beck](https://www.goodreads.com/book/show/387190.Test_Driven_Development)    
* [Growing Object-Oriented Software, Guided by Tests - Steve Freeman, Nat Pryce](https://www.goodreads.com/book/show/4268826-growing-object-oriented-software-guided-by-tests)    
* [Shameless green, chapter 1 of 99 bottles of OOP, Sandy Metz](https://www.goodreads.com/book/show/31183020-99-bottles-of-oop)  
* [Test contravariance - Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2017/10/03/TestContravariance.html)  
* ["Testing shows the presence, not the absence of bugs" - Edsger W. Dijkstra](https://blog.cleancoder.com/uncle-bob/2016/06/10/MutationTesting.html)  
* [London vs Chicago school, contributing-tests wiki - Justin Searls](https://github.com/testdouble/contributing-tests/wiki/London-school-TDD)  
* [Need driven development, Mock Roles, not Objects - Steve Freeman, Nat Pryce, Tim Mackinnon, Joe Walnes](http://jmock.org/oopsla2004.pdf)  
* [London vs Chicago - Robert C. Martin, Sandro Mancuso](https://cleancoders.com/series/comparativeDesign)  
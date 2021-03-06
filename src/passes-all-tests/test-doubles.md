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


As a final note, test doubles are not used just for unit tests, but throughout the [whole pyramid](test-pyramid.html).

#### Recommended reads
* [Mocks aren't stubs](https://martinfowler.com/articles/mocksArentStubs.html)  
* [Only mock type that you own, chapter 8 of Growing Object-Oriented Software, Guided by Tests - Steve Freeman, Nat Pryce](https://www.goodreads.com/book/show/4268826-growing-object-oriented-software-guided-by-tests)
* [Don't mock what you don't own, contributing-tests wiki - Justin Searls](https://github.com/testdouble/contributing-tests/wiki/Don%27t-mock-what-you-don%27t-own)

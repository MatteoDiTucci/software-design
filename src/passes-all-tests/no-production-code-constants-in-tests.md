## No production code constants in tests
When writing tests, we might be tempted to use production code constants to remove duplication. This creates a 
conflict of interests by which a test is defined using part of the code that is supposed to test. Such a situation 
can lead to bugs going unnoticed like in the following example.

(In Kotlin, `companion object` is the way to define constants known at compile time, the equivalent of `static` in Java)

```kotlin
class Greeter {

  fun greet(name: String): String {
    return "$DEFAULT_GREETING $name"
  }

  companion object {
    const val DEFAULT_GREETING: String = "Hello"
  }
}

@Test
fun `greet by name`() {
  val greeter = Greeter()

  val result = greeter.greet("Andrea")

  assertEquals("${Greeter.DEFAULT_GREETING} Andrea", result)
}
```

The problem with the above code is that if by mistake we change the value of `DEFAULT_GREETING` from `"Hello"` to `"Hellozxhgj"`
the test will still pass as it would be expecting `"Hellozxhgj Andrea"`. To catch this kind of bugs is 
enough not to use the production constant `DEFAULT_GREETING` in the test like follows.

```kotlin
@Test
  fun `greet by name`() {
    val greeter = Greeter()

    val result = greeter.greet("Andrea")

    assertEquals("Hello Andrea", result)
  }
```


<br/>  

[Teach me back ❤️](/introduction/introduction.html#teach-me-back)
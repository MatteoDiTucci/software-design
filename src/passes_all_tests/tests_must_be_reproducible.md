## Test must be reproducible
A test must not change throughout different executions, otherwise we would be running a different test every time.
The risk is ending up with tests that fail without apparent reason or worse with tests passing despite the presence of bugs. 
Let's consider the following example.

```kotlin
class AnnualSubscription(private val validityYear: Int) {

  fun isValid(currentDate: LocalDate): Boolean {
    return validityYear == currentDate.year
  }
}

@Test
fun `is valid during the year the subscription was made`() {
    val book = AnnualSubscription(2021)

    val result = book.isValid(LocalDate.now()) // today is the 3rd of March 2021

    assertTrue(result)
}
```

The above test always passes during 2021. However, it will start failing from the 1st of January 2022 because of `LocalDate.now()`.
To make the test deterministic we have to replace `LocalDate.now()` with a constant date like follows.

```kotlin
@Test
  fun `is valid during the year the subscription was made`() {
    val book = AnnualSubscription(2021)

    val result = book.isValid(LocalDate.of(2021, Month.MARCH, 3))

    assertTrue(result)
  }
```

<br/>

#### Recommended reads
* [Purely functional random number generation, chapter 6 of Functional Programming in Scala - Rúnar Bjarnason, Paul Chiusano, Tony Morris](https://www.goodreads.com/book/show/13541678-functional-programming-in-scala)
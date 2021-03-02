## Test coverage is not enough
Test coverage measures how much code in a codebase is invoked when we run all tests. However, it is not a measure of quality
of the tests. This means a codebase with 100% of coverage can still be plenty of bugs. Let's take as example the following code.

```kotlin
data class Bill(private val amount: Int) {

  fun split(parts: Int): Bill {
    return Bill(amount / parts)
  }
}

@Test
fun `split the bill in equal parts for 4 people`() {
  val bill = Bill(100)

  val result = bill.split(4)

  assertEquals(Bill(25), result)
}
```

(In Kotlin the `data class` keyword makes sure that two instances of `Bill` are equal if their `amount` is equal).

The above code has 100% test coverage. However, we could break it by simply calling `split(0)` over `Bill`.
For this reason, it is important to test against as many inputs as possible. For instance, in the above example this means
trying different input parameters for the `Bill` constructor and for the `split` method. Defining one test for each possible
combination of inputs is not feasible, but there are some testing methodologies that let us test for many inputs in very little time:
* Parameterized testing: execute one test multiple times specifying different input parameters
* Property based testing: execute one test multiple times specifying which kind of inputs are valid and letting a library randomly generate them

<br/>  

#### Recommended reads
* [Parameterised testing with Junit in Kotlin - Baeldung](https://www.baeldung.com/parameterized-tests-junit-5)  
* [Jqwik, a property based testing library in Java](https://jqwik.net/)  
* [Fuzz testing](https://en.wikipedia.org/wiki/Fuzzing)    
* [Formal verification](https://en.wikipedia.org/wiki/Formal_verification) 
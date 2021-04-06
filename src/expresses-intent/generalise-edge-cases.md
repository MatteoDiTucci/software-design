## Generalise edge cases

Generalising edge cases makes code easier to understand as we have a single implementation to reason on. Let's assume
we need to implement the following specifications for an airport check-in application:
* Verify a passenger passport has not expired
* If the passenger is a minor from a national flight, verify in addition one of their parents passport
* If the passenger is a minor from an international flight, verify in addition both parents passports

```kotlin
class CheckIn(private val today: LocalDate) {

  fun allowBoarding(passport: Passport): Boolean {
    return passport.isValid(today)
  }

  fun allowNationalFlightMinorBoarding(passport: Passport,
                                       parentPassport: Passport): Boolean {
    return passport.isValid(today) && parentPassport.isValid(today)
  }

  fun allowInternationalFlightMinorBoarding(passport: Passport,
                                            firstParentPassport: Passport,
                                            secondParentPassport: Passport): Boolean {
    return passport.isValid(today)
        && firstParentPassport.isValid(today)
        && secondParentPassport.isValid(today)
  }
}

class Passport(private val name: String,
               private val lastName: String,
               private val expirationDate: LocalDate) {

  fun isValid(today: LocalDate): Boolean {
    return expirationDate.isBefore(today)
  }
}
```

The above `CheckIn` class can be simplified by collapsing the three methods into one that accepts a list of `Passport` 
like follows.

```kotlin
class CheckIn(private val today: LocalDate) {

  fun allowBoarding(passports: List<Passport>): Boolean {
    if (passports.isEmpty()){
      return false
    }
      
    // In Kotlin, all() returns true only if all passports are valid
    return passports.all { passport -> passport.isValid(today) }
  }
}

class Passport(private val name: String,
               private val lastName: String,
               private val expirationDate: LocalDate) {

  fun isValid(today: LocalDate): Boolean {
    return expirationDate.isBefore(today)
  }
}
```

The above version of `CheckIn` is easier to understand in terms of both internal implementation and the exposed methods. 
However, we could argue that the code is not expressing anymore the need to verify extra passports in case of minors. 
That information is still contained in the unit tests of the `CheckIn` class, some of which might be as follows.

```kotlin
  @Test
  fun `blocks national flight boarding of a minor if their parent passport has expired`() {
    val checkIn = CheckIn(LocalDate.of(2021, 4, 4))
    val passport = Passport("Andrea", "Diga Smith", LocalDate.of(2025, 1, 1))
    val parentPassport = Passport("Michael", "Diga", LocalDate.of(2000, 1, 1))


    assertFalse(checkIn.allowBoarding(listOf(passport, parentPassport)))
  }

  @Test
  fun `blocks international flight boarding of a minor if one of their parent passport has expired`() {
    val checkIn = CheckIn(LocalDate.of(2021, 4, 4))
    val passport = Passport("Andrea", "Diga Smith", LocalDate.of(2025, 1, 1))
    val firstParentPassport = Passport("Michael", "Diga", LocalDate.of(2000, 1, 1))
    val secondParentPassport = Passport("Mark", "Smith", LocalDate.of(2035, 1, 1))

    assertFalse(checkIn.allowBoarding(listOf(
                                        passport,
                                        firstParentPassport,
                                        secondParentPassport)))
  }
```

<br/>  

#### Recommended reads
* [Following the Flocking Rules, chapter 3.6 of 99 bottles of OOP - Sandy Metz](https://www.goodreads.com/book/show/31183020-99-bottles-of-oop)

#### Teach me back
I really appreciate any [feedback]((/introduction/introduction.html#teach-me-back)) about the book and my current understanding of software design.
## Generalise edge cases

Generalising edge cases makes code easier to understand as we have a single implementation to reason on. Let's assume
we need to implement the following specifications for an airport check-in application:
* Verify a passenger passport has not expired
* If the passenger is a minor, verify that the accompanying parent passport has not expired

```kotlin
class CheckIn(private val today: LocalDate) {

  fun checkPassport(passport: Passport): Boolean {
    return passport.isValid(today)
  }

  fun checkMinorPassport(minorPpassport: MinorPassport):Boolean {
    return minorPpassport.isValid(today)
  }
}

class Passport(private val name: String,
               private val lastName: String,
               private val expirationDate: LocalDate) {

  fun isValid(today: LocalDate): Boolean {
    return expirationDate.isBefore(today)
  }
}

class MinorPassport(private val name: String,
                    private val lastName: String,
                    private val expirationDate: LocalDate,
                    private val parentPassport: Passport) {

  fun isValid(today: LocalDate): Boolean {
    return expirationDate.isBefore(today) && parentPassport.isValid(today)
  }
}
```

The above `CheckIn` class can be simplified by generalising `MinorPassport` to `Passport` like follows.

```kotlin
class CheckIn(private val today: LocalDate) {

  fun checkPassport(passport: Passport): Boolean {
    return passport.isValid(today)
  }
}

open class Passport(private val name: String,
                    private val lastName: String,
                    private val expirationDate: LocalDate) {

  open fun isValid(today: LocalDate): Boolean {
    return expirationDate.isBefore(today)
  }
}

class MinorPassport(private val name: String,
                    private val lastName: String,
                    private val expirationDate: LocalDate,
                    private val parentPassport: Passport) : Passport(name, lastName, expirationDate) {

  override fun isValid(today: LocalDate): Boolean {
      // The keyword super is used to invoke the isValid method of Passport
      return super.isValid(today) && parentPassport.isValid(today) 
  }
}
```

(In Kotlin you need to mark a class or a method with the keyword `open` if you want to extend it)

The above code differs from the original one for three aspects:
* `MinorPassport` extends `Passport`
* `MinorPassport` overrides the method `isValid` of `Passport`, checking the parent passport expiration date too
* `CheckIn` exposes a single `checkPassport` method that can be used for both adults and minors passports

Now the `CheckIn` class is easier to understand as it only deals with checking a passport. Furthermore, we removed 
knowledge duplication in `MinorPassport`.

Inheritance is not the only way to generalise edge cases. Another common approach is to use a collection 
instead of handling multiple scenarios that only differ by the occurrences of something.

<br/>  

#### Recommended reads
* [Following the Flocking Rules, chapter 3.6 of 99 bottles of OOP - Sandy Metz](https://www.goodreads.com/book/show/31183020-99-bottles-of-oop)

#### Teach me back
I really appreciate any [feedback]((/introduction/introduction.html#teach-me-back)) about the book and my current understanding of software design.
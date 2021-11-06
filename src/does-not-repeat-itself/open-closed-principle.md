## Open-closed principle

A good sign of low knowledge duplication is the creation of new classes instead of modifying existing ones when we implement 
new specifications. To give an example, let's assume we need to implement the following:
* A light bulb that returns the message "I am shining"

```kotlin
class LightBulb {

  fun lightUp(): String {
    return "I am shining"
  }
}
```

So far so good. Now let's assume we receive further specifications:
* A light bulb that returns the message "I am shining" if it is on
* A light bulb that returns the message "I am dark" if it is off

It does not seem a big deal, so we could proceed modifying the existing `LightBulb` class like the following:
```kotlin
enum class LightBulbState {
  ON,
  OFF
}

class LightBulb(private val state: LightBulbState) {

  fun lightUp(): String {
    if (state == LightBulbState.ON) {
      return "I am shining"
    }
    return "I am dark"
  }
}
```

Let's now implement one last specification:
* A light bulb that returns the message "I am broken" if it is broken

If we insist modifying the `LightBulb` class we end up like the following:

```kotlin
enum class LightBulbState {
  ON,
  OFF,
  BROKEN
}

class LightBulb(private val state: LightBulbState) {

  fun lightUp(): String {
    if (state == LightBulbState.ON) {
      return "I am shining"
    }
    if (state == LightBulbState.OFF) {
      return "I am dark"
    }
    return "I am broken"
  }
}
```

If this trend of specifications continues, we will end up having a very long `lightUp` method and many unit tests just 
for it. The result will be `LightBulb` not clearly expressing its intent as well as being a hot spot containing too much
knowledge. To prevent this we can spread the `lightUp` knowledge across the different values of `LightBulbState`
like follows.

```kotlin
enum class LightBulbState {
  ON {
      override fun lightUp(): String {
          return "I am shining"
      }
  },
  OFF {
      override fun lightUp(): String {
          return "I am dark"
      }
  },
  BROKEN {
      override fun lightUp(): String {
          return "I am broken"
      }
  };

    abstract fun lightUp(): String
}

class LightBulb(private val state: LightBulbState) {

  fun lightUp(): String {
      return state.lightUp()
  }
}
```

Now each `LightBulbState` is responsible for its bit of `lightUp` knowledge while the `LightBulb` class remains short and clear to understand.
This means the following specification can be implemented by adding another `LightBulbState`, without modifying `LightBulb`:
* A light bulb that returns the message "I am shining softly" if it is dimmed.

```kotlin
enum class LightBulbState {
  ON {
      override fun lightUp(): String {
          return "I am shining"
      }
  },
  OFF {
      override fun lightUp(): String {
          return "I am dark"
      }
  },
  BROKEN {
      override fun lightUp(): String {
          return "I am broken"
      }
  },
  DIMMED {
      override fun lightUp(): String {
          return "I am shining softly"
      }
  };

    abstract fun lightUp(): String
}

class LightBulb(private val state: LightBulbState) {

  fun lightUp(): String {
      return state.lightUp()
  }
}
```

<br/>

#### Recommended reads
* [Starting With the Open/Closed Principle, chapter 3.2 of 99 bottles of OOP - Sandy Metz](https://www.goodreads.com/book/show/31183020-99-bottles-of-oop)
* [OCP: The Open-Closed Principle, chapter 9 Agile Software Development, Principles, Patterns, and Practices  - Robert C. Martin](https://www.goodreads.com/book/show/84985.Agile_Software_Development_Principles_Patterns_and_Practices)
* [The open-closed principle, chapter 8 of Clean Architecture - Robert C. Martin](https://www.goodreads.com/book/show/18043011-clean-architecture)
* [Why you shouldn’t use booleans in REST APIs - Hany Elemary](https://medium.com/geekculture/why-you-shouldnt-use-booleans-in-rest-apis-2747b187876c)

#### Teach me back
I really appreciate any [feedback]((/introduction/introduction.html#teach-me-back)) about the book and my current understanding of software design.
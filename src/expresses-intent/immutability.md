## Immutability
Understanding a piece of code is harder if objects can change their state. Let's take a look at the following example.

```kotlin
enum class LightBulbState {
  ON {
    override fun lightUp(): String = "bright"
  },
  OFF {
    override fun lightUp(): String = "dark"
  };

  abstract fun lightUp(): String
}

class LightBulb(private var state: LightBulbState) {

  fun lightUp(): String {
    return state.lightUp()
  }

  fun setState(newLightBulbState: LightBulbState) {
    state = newLightBulbState
  }
}

class Room {
  private val lightBulb: LightBulb

  constructor(lightBulb: LightBulb) {
    lightBulb.setState(LightBulbState.OFF)
    this.lightBulb = lightBulb
  }
}

class App {

  fun main() {
    val lightBulb = LightBulb(LightBulbState.ON)
    val room = Room(lightBulb)

    val result = "The room is ${lightBulb.lightUp()}"
  }
}
```

Just looking at the `main` method of `App`, we would expect `result` to be `The room is bright`. However, because of the
`Room` constructor, the actual content of `result` is `The room is dark`. To avoid this bad surprise, `LightBulb` can become 
immutable, meaning its state cannot be modified. Let's take a look how we can achieve this.

```kotlin
enum class LightBulbState {
    ON {
        override fun lightUp(): String = "bright"
    },
    OFF {
        override fun lightUp(): String = "dark"
    };

    abstract fun lightUp(): String
}

class LightBulb(private val state: LightBulbState) {

  fun lightUp(): String {
    return state.lightUp()
  }

  fun setState(newLightBulbState: LightBulbState): LightBulb {
    return LightBulb(newLightBulbState)
  }
}

class Room {
  private val lightBulb: LightBulb

  constructor(lightBulb: LightBulb) {
    this.lightBulb = lightBulb.setState(LightBulbState.OFF)
  }
}

class App {

  fun main() {
    val lightBulb = LightBulb(LightBulbState.ON)
    val room = Room(lightBulb)

    val result = "The room is ${lightBulb.lightUp()}"
  }
}

App().main()
```

The above code differs from the original one for three aspects:
* The `setState` method of `LightBulb` does not modify the state, instead it returns a brand-new instance of `LightBulb`
* The constructor of `Room` stores the brand-new `LightBulb` returned by the invocation of `setState`, instead of the `LightBulb` passed as a constructor parameter
* The `LightBulb` constructor uses `val` instead of `var`: in Kotlin a variable defined as `val` cannot be re-assigned

Now the content of result in the `main` method of `App` is what we would expect from the beginning: `The room is bright`.
Immutability makes it easier to reason about a piece of code and it is a game changer when [concurrency](https://en.wikipedia.org/wiki/Concurrency_(computer_science))
comes into play. To obtain immutability we need to:
* return a new instance of an object instead of modifying its state
* prevent the re-assignment of variables

Unfortunately it can be tricky to write immutable code. Let's take a look at the following piece of example.

```kotlin
class Shelf(private val books: List<String>) {

  fun numbersOfBooks(): Int {
    return books.size
  }
}

class App {

  fun main() {
    val books = mutableListOf("The Secret Adversary", "The Big Four")
    val shelf = Shelf(books)

    books.add("Giant's Bread")

    val result = "The shelf contains ${shelf.numbersOfBooks()} books"
  }
}
```

The content of `result` in the `main` method of `App` will be `The shelf contains 3 books`. In fact, even if `Shelf` uses
`val` and does not allow its state to change, the `books` list can still be modified by code outside `Shelf`. For `Shelf`
to be immutable, it has to make a local copy of the `books` list received in the constructor as follows.

```kotlin
class Shelf {
  private val books: List<String>

  constructor(booksForShelf: List<String>) {
    books = booksForShelf.toList() // make an exact copy
  }

  fun numbersOfBooks(): Int {
    return books.size
  }
}

class App {

  fun main() {
    val books = mutableListOf("The Secret Adversary", "The Big Four")
    val shelf = Shelf(books)

    books.add("Giant's Bread")

    val result = "The shelf contains ${shelf.numbersOfBooks()} books"
  }
}
```

The content of `result` in the `main` method of `App` is now `The shelf contains 2 books`.  
As a final note, reasoning about state is hard and that's why it is worth deepen the paradigm of functional programming
which removes the concept of state as much as possible. 

#### Recommended reads
* [Appreciating Immutability, section 5.3 of 99 bottles of OOP - Sandy Metz](https://www.goodreads.com/book/show/31183020-99-bottles-of-oop)
* [Practical Function Programming in Scala - Gabriel Volpe](https://leanpub.com/pfp-scala)
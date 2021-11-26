## Dependency inversion principle

Code that changes frequently should depend on code that rarely changes. This is important to minimises the amount of code
we need to adjust when implementing new specifications. Let's take a look at the following example.

```kotlin
class Transfer(private val sender: String,
               private val receiver: String,
               private val amount: Int) {

  fun save(transferFile: TransfersFile) {
    transferFile.save(sender, receiver, amount)
  }
}

class TransferFile {

    fun save(sender: String, receiver: String, amount: Int) {
        File("transfers.csv").writeText("$sender,$receiver,$amount")
    }
}

class App {

    fun main() {
        val transfer = Transfer("Andrea", "Dani", 100)
        val file = TransferFile()
        transfer.save(file)
    }
}
```

In the above code, the class `Transfer` depends on `TransferFile` as the former takes the latter as input parameter
in the `save` method. Now let's assume we know that the class `TransferFile` is going to change a lot in the future, 
for instance because it might become a different class that saves `Transfer` to a database or because it will connect
to a third party api over the internet and its `save` method signature might change. Even worse, we won't be able to modify `TransferFile`
in the future as another developer is going to take care of it. In this situation, the risk is that everytime `TransferFile`
changes, we will be forced to change our `Transfer` class as well. We can prevent this by leveraging the dependency inversion
principle like in the following code.

```kotlin
interface TransferStorage {
  fun save(sender: String, receiver: String, amount: Int)
}

class Transfer(private val sender: String, 
                    private val receiver: String, 
                    private val amount: Int) {

  fun save(file: TransferStorage) {
    file.save(sender, receiver, amount)
  }
}

class TransferFile : TransferStorage {

  override fun save(sender: String, receiver: String, amount: Int) {
    File("transfers.csv").writeText("$sender,$receiver,$amount")
  }
}

class App {

  fun main() {
    val transfer = Transfer("Andrea", "Dani", 100)
    val file = TransferFile()

    transfer.save(file)
  }
}
```

The above code differs from the original one for three aspects:
* We introduced `TransferStorage` which contains an abstract definition of the method `save`
* `Transfer` does not depend anymore on `TransferFile` as it now depends on `TransferStorage`
* `TransferFile` now depends on `TransferStorage` as it implements its `save` method

With this, we have inverted the dependency we had from `Transfer` to `TransfersFile`. In fact, `Transfer` has become unaware
of any change that might happen to `TransfersFile`. Furthermore, `Transfer` becomes in general unaware of the technology
used to store it: as far as a class implements `TransferStorage`, it can be passed to `Transfer` 
inside the `main` method of `App`. This means we could be using a new class `TransfersDatabase` and `Transfer` would not know it.
On the contrary, `TransfersFile` now depends on `TransferStorage` but that's fine as we know `TransferStorage` will rarely
change.  

As a final note, `TransferStorage` has also become the [single authoritative representation](one-single-authoritative-representation.html)
of how to store `Transfer`.

<br/>

#### Recommended reads
* [Depending on abstractions, chapter 4.9 of 99 bottles of OOP - Sandy Metz](https://www.goodreads.com/book/show/31183020-99-bottles-of-oop)
* [The dependency inversion principle, chapter 11 of Clean Architecture - Robert C. Martin](https://www.goodreads.com/book/show/18043011-clean-architecture)
* [Dependency injection, chapter 11 of Clean Code - Robert C. Martin](https://www.goodreads.com/book/show/3735293-clean-code)
* [The onion architecture - Jeffrey Palermo](https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/)

#### Teach me back
I really appreciate any [feedback](../introduction/introduction.html#teach-me-back) about the book and my current understanding of software design.
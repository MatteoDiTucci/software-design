## If testing is hard, inject what you need to verify
When you have a hard time testing something, the solution is usually to inject the thing you would like to verify.  
Suppose you have a `Car` class storing passengers by their name.

```kotlin
class Car {
  private val passengers: MutableSet<String> = HashSet()
    
  fun storePassenger(name: String) {
    passengers.add(name)
  }
}
```

How to test that the method `storePassenger(name: String)` stores a name into the `passengers` set?
The typical solution is to define another method in `Car` to check if it contains a passenger.

```kotlin
class Car {
  private val passengers: MutableSet<String> = HashSet()
    
  fun storePassenger(name: String) {
    passengers.add(name)
  }
       
  fun containsPassenger(name: String): Boolean { 
    return passengers.contains(name)
  }
}
```

So we can write the following test

```kotlin
@Test
fun `stores the names of the passengers`() {
  val car = Car()
  
  car.storePassenger("Andrea")

  assertTrue(passengers.containsPassenger("Andrea"))
}
```

However, this is already a disappointment because we are forced to write a public method just for the sake of testing.
Moreover, what if by specifications we must prevent any other code to query `Car` about its passengers?
The solution is to inject the `passengers` set a construction time.

```kotlin
class Car(private val passengers: MutableSet<String>) {

  fun storePassenger(name: String) {
    passengers.add(name)
  }
}
```

Now we can test like follows.

```kotlin
@Test
fun `stores the names of the passengers`() {
  val passengers = HashSet()
  val car = Car(passengers)
  
  car.storePassenger("Andrea")

  assertTrue(passengers.contains("Andrea"))
}
```

Injecting the `passengers` set, we can get rid of the useless `containsPassenger` method. Furthermore, we have now the
opportunity to make our code more modular, making `Car` independent of the data structure used to store the passengers.
For this, we can use `MutableCollection<String>` instead of `MutableSet<String>`

```kotlin
class Car(private val passengers: MutableCollection<String>) {

  fun storePassenger(name: String) {
    passengers.add(name)
  }
}
```

[1] [Context independence section, chapter 6 of Growing Object-Oriented Software, Guided by Tests - Steve Freeman, Nat Pryce](https://www.goodreads.com/book/show/4268826-growing-object-oriented-software-guided-by-tests)
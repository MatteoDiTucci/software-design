## Deep and narrow classes
We are doomed to write convoluted code when interacting with classes that have poor public methods. On the contrary,
we are brought to write readable code when interacting with classes that have great public methods. Let's consider 
the following `play` public method for the [FizzBuzz game](https://en.wikipedia.org/wiki/Fizz_buzz).

```kotlin
class FizzBuzz {

  fun play(number: Int): String {
  
    if (number % 5 == 0 && number % 3 == 0)
      return "fizz buzz"
  
    if (number % 5 == 0)
      return "buzz"

    if (number % 3 == 0)
      return "fizz"

    return number.toString()
  }
}
```

The above signature of the `play` method leads to the following code when playing with the numbers between 1 and 100.

```kotlin
class App {

  fun main() {
 
    val fizzBuzz = FizzBuzz()    
    val result = ArrayList<String>()
    
    for (number in 1..100)
      result.add(fizzBuzz.play(number))
  }
}
```

Now, let's change the signature of the `play` method like follows.

```kotlin
class FizzBuzz {

  fun play(from: Int, to: Int): List<String> {
  
    val result = ArrayList<String>()
    
    for (number in from..to)
      result.add(playSingleNumber(number))
    
    return  result
  }

  private fun playSingleNumber(number: Int): String {
  
    if (number % 5 == 0 && number % 3 == 0)
      return "fizz buzz"
    
    if (number % 5 == 0)
      return "buzz"
    
    if (number % 3 == 0)
      return "fizz"
  
    return number.toString()
  }
}
```

The above signature of the `play` method leads to the following code when playing with the numbers between 1 and 100.

```kotlin
class App {

  fun main() {
  
    val fizzBuzz = FizzBuzz()
    val result = fizzBuzz.play(1, 100)        
  }
}
```

Now the `main` method is more readable. The for-loop has not disappeared, it just moved from `App` to `FizzBuzz`.  
However, such a shift becomes remarkable if applied to a codebase with many classes:
* In the first approach, the for-cycle is repeated every time a piece of code interacts with `FizzBuzz`. In the second
  approach, we are guaranteed the for-cycle is written only once: inside `FizzBuzz`.
* If `main` interacted with 4 classes each one using the first approach of `FizzBuzz`, `main` would contain 4 for-loops.
  If the 4 classes were to use the second approach, `main` will contain zero for-loops.

To summarise with a catchphrase from John Ousterhout, classes should be narrow and deep:
* narrow means few public methods with few input parameters
* deep means public methods get a lot of things done for the caller

<br/>  

#### Recommended reads
* ["Bad programmers worry about the code. Good programmers worry about data structures and their relationships" - Linus Torvalds](https://lwn.net/Articles/193245/)  
* ["Show me your tables, and I won't usually need your flowcharts; they'll be obvious.", chapter 9 of The Mythical Man-Month - Fred Brooks](https://www.goodreads.com/book/show/13629.The_Mythical_Man_Month)  
* [ava and Unix I/O, section 4.7 of A Philosophy of Software Design - John Ousterhout](https://www.goodreads.com/en/book/show/39996759-a-philosophy-of-software-design)  
* [A web of objects, chapter 2 of Growing Object-Oriented Software, Guided by Tests - Steve Freeman, Nat Pryce](https://www.goodreads.com/book/show/4268826-growing-object-oriented-software-guided-by-tests)  
* [Choosing Names, section 2.8 of 99 bottles of OOP - Sandy Metz](https://www.goodreads.com/book/show/31183020-99-bottles-of-oop)

#### Teach me back
I really appreciate any [feedback](../introduction/introduction.html#teach-me-back) about the book and my current understanding of software design.
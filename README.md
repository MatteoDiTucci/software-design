# Introduction

### Why read this book
Software design is a craft not a science. To master it you cannot just study a book that explains comprehensively all the theory behind it.
You have to read the literature of many great developers, looking for advices that repeat: that's what is currently considered good software design.
However, the literature is vast, and it takes much time to get the big picture out of single sources.
This book makes three promises:
  * to be a catalogue of everything that constitutes good software design
  * to be very succinct
  * to provide trustworthy sources to go deeper 

### Who should read this book
The book is intended mainly for people who just started their journey into software design or who are in the middle of it.
If you have being seriously studying the topic for 5+ years, likely you are not going to find big surprises. However, I still 
suggest you to go through the table of contents: you might find one or two learnings worth a quick read. 
As a last consideration, the book is heavily skewed towards object-oriented programming though many advices apply to all [programming paradigms](http://www.cs.albany.edu/~sdc/CSI500/Downloads/ProgrammingParadigmsVanRoyChapter.pdf).
Code examples are written in Kotlin, but they are as basic as possible so you do not need any prior knowledge.

### Who I am
My name is Matteo Di Tucci, I currently work at ThoughtWorks.  
I picked up coding in university because I did not know what to do when growing up.
I still do not know, but along the way I pleasantly discovered I like software and its design.

### Teach me back
I really appreciate any feedback about the book and my current understanding of software design.
You can share any feedback by creating a pull request.
If you do not know what a pull request is, take a look [here](https://guides.github.com/activities/hello-world/#:~:text=Pull%20Requests%20are%20the%20heart,merge%20them%20into%20their%20branch.&text=You%20can%20even%20open%20pull,repository%20and%20merge%20them%20yourself.).

### Simple code
It is hard to define what good design looks like: nobody was able to come up with an unambiguous definition so far. 
The thing that came closer to that is the [C2 Wiki](https://wiki.c2.com/?XpSimplicityRules) simple code rules.  
Simple code:

* Passes all tests
* Expresses intent
* Does not repeat itself
* Does not contain superfluous parts

If a piece of code respects all those 4 rules then we can consider it good code.  
This book is divided in 4 sections: one for each the simple code rules.

# Tables of contents

### Passes all tests
##### [Test pyramid](#Test pyramid)
#### [If testing is hard, inject what you need to check](#If-testing-is-hard,-inject-what-you-need-to-check)
#### Mock vs stub vs spy
#### Test naming
#### Test coverage is not enough: parameterised tests
#### Tests must be reproducible
^ no Math.random() nor LocalDate.now()
#### Do not test libraries
#### Dry vs moist tests
#### Test driven design (TDD)
^ Here mention sources to Outside-in vs inside-out TDD and Classical vs mockist TDD
#### Component tests vs end-to-end tests vs monitoring tradeoffs
#### Contract testing
#### Do not use production constants
#### Performance tests
#### Linting
### Expresses intent
#### [Naming](#Naming)
#### [Deep and narrow classes](#Deep-and-narrow-classes)
#### Small classes and short methods
#### SOLID principles
#### Usually composition is better than inheritance
#### Generalise edge cases
#### Immutability
#### Comments the why
#### Visual indentation
^ Kevlin Henney talk
#### Folder structure
#### Encapsulation
### Does not repeat itself
#### One single authoritative knowledge representation
#### Do not abstract by pattern matching
#### Polymorphism
### Does not contain superfluous parts
#### If you need it tomorrow, then you need it
^ Here discuss about obvious implementation and how playing it dumb can lead to substantial reworking when for instance
the definition of a public interface is split between 2 user stories: happy path and sad paths
#### Wrong abstraction is more costly than duplication
#### Do not abuse design patterns

# Passes all tests

### Test pyramid
There are different kinds of tests, each one meant for a different purpose. When describing them below, the expression
_external dependencies_ means anything that is reached over the network like a database, another team REST endpoint, a queue, 
third party apis, etc. For _service_ instead, we mean a bunch of code deployed as a whole.  

* **Acceptance tests**  
  A feature behaves as expected across all service layers (e.g. back end and front end) or even
  across different services. External dependencies are replaced through libraries like LocalStack or Wiremock.
  In a web application for instance, acceptance tests are defined with tools like Selenium or Cypress. 
* **Functional tests**  
  A feature behaves as expected considering a single layer of a service. For example, if a service have both back end and 
  front end, there will be distinct component tests for the back end and front end. External dependencies are replaced either 
  by libraries like LocalStack or by [doubles](#Mock-vs-stub-vs-spy). Functional tests are also called component tests.
* **Integration tests**  
  The service integrates correctly with external dependencies. External dependencies are replaced by libraries like 
  LocalStack or Wiremock. If you use code [doubles](#Mock-vs-stub-vs-spy) for the external dependencies then it is a unit test.
* **Unit tests**  
  A class behaves as expected. Unit tests are most valuable when testing business logic: if a class is just a 
  [delegator](https://en.wikipedia.org/wiki/Delegation_pattern) or just coordinates other classes (e.g. onion architecture use cases),  
  do not use unit tests as functional tests already provide coverage.
  If the class under test uses other classes whose construction is cumbersome, those can be [doubles](#Mock-vs-stub-vs-spy)  


The above list is ordered by how much time a test takes to execute, from the slowest (acceptance) to the fastest (unit). 
For this reason, it is recommended to have a pyramid of tests: a handful of acceptance tests, some functional and infrastructure
tests and many unit tests. In particular:

* **Acceptance tests**  
  Only for default uses of a feature
* **Component tests (also known as functional tests)**  
  For both default and exceptional-erroneous uses of a feature
* **Integration tests**  
  For both default and exceptional-erroneous integrations with external dependencies
* **Unit tests**  
  For both default and exceptional-erroneous usage of a single class
  

[1] [Test Pyramid (in short) - Martin Fowler](https://martinfowler.com/bliki/TestPyramid.html)  
[2] [Test Pyramid (in depth) - Ham Vocke](https://martinfowler.com/articles/practical-test-pyramid.html)  
[3] [Growing Object-Oriented Software, Guided by Tests - Steve Freeman, Nat Pryce](https://www.goodreads.com/book/show/4268826-growing-object-oriented-software-guided-by-tests)  
[4] ["Testing shows the presence, not the absence of bugs" - Edsger W. Dijkstra](https://blog.cleancoder.com/uncle-bob/2016/06/10/MutationTesting.html)  



### If testing is hard, inject what you need to check
When you have a hard time testing something, the solution is usually to inject the thing you would like to check.  
Suppose you have a car class storing passengers by their name.

```
class Car {
  private val passengers: MutableSet<String> = HashSet()
    
  fun addPassenger(name: String) =
    passengers.add(name)
}
```

How to test that the method `addPassenger(name: String)` stores a name into the passenger set?
The typical solution is to define a method to query the car about its passengers.

```
class Car {
  private val passengers: MutableSet<String> = HashSet()
    
  fun addPassenger(name: String) =
    passengers.add(name)
       
  fun containsPassenger(name: String) = 
    passengers.contains(name)
}
```

So we can write the following test

```
@Test
fun 'stores the names of the passengers'() {
  val car = Car()
  
  car.addPassenger("Andrea")

  assertTrue(passengers.containsPassenger("Andrea"))
}
```

However, this is already a disappointment because we are forced to write a public method just for the sake of testing.  
Moreover, what if we must prevent any other code
to query the passengers? The solution is to inject the passenger set a construction time.

```
class Car(private val passengers: MutableSet<String>) {

  fun addPassenger(name: String) =
    passengers.add(name)
}
```

Now we can test like follows

```
@Test
fun 'stores the names of the passengers'() {
  val passengers = HashSet()
  val car = Car(passengers)
  
  car.addPassenger("Andrea")

  assertTrue(passengers.contains("Andrea"))
}
```

Injecting the passengers set led to two benefits:
* No public method needs to be defined just for testing purposes
* The class becomes independent of the data structure used to store the passengers, making the code more modular

[1] [Context independence section, chapter 6 of Growing Object-Oriented Software, Guided by Tests - Steve Freeman, Nat Pryce](https://www.goodreads.com/book/show/4268826-growing-object-oriented-software-guided-by-tests)


# Expresses intent
### Naming
Great naming let us understand what the code means without inspecting its details. In fact, we should be able 
to understand an entire codebase just reading the names of classes, methods and variables, never looking at what they contain.
This clarity is reached by naming classes, methods and variables after what they do instead of how they do it.
The how can be looked up later if more details are needed.   
The benefits are two:
* names do not contain any noise 
* if implementation changes, the name remains relevant

Let's consider the following class

```
class BiDimensionalCoordinates(private val x: Int, private val y: Int) {
  
  fun sumByXandY(addend: BiDimensionalCoordinates): BiDimensionalCoordinates {
    return BiDimensionalCoordinates(x + addend.x, y + addend.y)
  }
}
```

In the above code, both class and method names leak the how. Given the distinction we made about the what and the how, 
we can ask two questions. The first one is: if we remove the how from the `BiDimensionalCoordinates` names, does the class still express intent?

```
class Coordinates(private val x: Int, private val y: Int)
  
  fun sum(addend: Coordinates): Coordinates {
    return Coordinates(x + addend.x, y + addend.y)
  }
}
```

The answer is yes, as the above class still expresses clear intent, but without bothering the reader with the noise of the how.
The second questions is: would the naming in `BiDimensionalCoordinates` still make sense if we were to switch to three-dimensional coordinates?

```
class BiDimensionalCoordinates(private val x: Int, private val y: Int, private val z: Int) {
  
  fun sumByXandY(addend: BiDimensionalCoordinates): BiDimensionalCoordinates {
    return BiDimensionalCoordinates(x + addend.x, y + addend.y, z + addend.z)
  }
}
```

The answer is no. `BiDimensionalCoordinates` does not express its intent anymore as it is lying to the reader: the names suggest a two dimensions representation
when it is actually three. However, the `Coordinates` class would still express its intent as shown below.

```
class Coordinates(private val x: Int, private val y: Int, private val z: Int) {
  
  fun sum(addend: Coordinates): Coordinates {
    return Coordinates(x + addend.x, y + addend.y, z + addend.z)
  }
}
```


[1 Names, section 1.1.1 of 99 bottles of OOP - Sandy Metz](https://www.goodreads.com/book/show/31183020-99-bottles-of-oop)  
[2 Choosing Names, section 2.8 of 99 bottles of OOP - Sandy Metz](https://www.goodreads.com/book/show/31183020-99-bottles-of-oop)  
[3 Meaningful Names, chapter 2 of Clean Code - Robert C. Martin](https://www.goodreads.com/book/show/3735293-clean-code)  
[4 Developing a Ubiquitous Language, chapter 2 of Domain-Driven Design Distilled - Vaughn Vernon](https://www.barnesandnoble.com/w/domain-driven-design-distilled-vaughn-vernon/1124175630)  


### Deep and narrow classes
We are doomed to write convoluted code when interacting with classes that have poor public methods. On the contrary, 
we are brought to write readable code when interacting with classes that have great public methods.  
Let's consider the following `play` public method for the [FizzBuzz game](https://en.wikipedia.org/wiki/Fizz_buzz).
```
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

```
class App {

  fun main() {
 
    val fizzBuzz = FizzBuzz()    
    val result = ArrayList<String>()
    
    for (number in 1..100)
      result.add(fizzBuzz.play(number))
  }
}
```

Now, let's change the signature of the `play` method like it follows.

```
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

```
class App {

  fun main() {
  
    val fizzBuzz = FizzBuzz()
    val result = fizzBuzz.play(1, 100)        
  }
}
```

Now the code in `main` is more readable. Complexity (the for-loop) has not disappeared, it just moved from `App` to `FizzBuzz`.  
However, such a shift becomes remarkable if applied to a codebase with many classes:
* In the first approach, the for-cycle is repeated every time a piece of code interacts with `FizzBuzz`.  
  In the second approach, we are guaranteed the for-cycle is written only once, inside `FizzBuzz`.
* If `main` interacted with 4 classes each one using the first approach of `FizzBuzz`, `main` would contain 4 for-loops.  
  If the 4 classes were to use the second approach, `main` will contain zero for-loops.

To summarise with a catchphrase, classes should be narrow and deep:
* narrow means few public methods with few input parameters
* deep means public methods handle as much complexity as possible for the caller

[1 "Bad programmers worry about the code. Good programmers worry about data structures and their relationships" - Linus Torvalds](https://lwn.net/Articles/193245/)  
[2 "Show me your tables, and I won't usually need your flowcharts; they'll be obvious.", chapter 9 of The Mythical Man-Month - Fred Brooks](https://www.goodreads.com/book/show/13629.The_Mythical_Man_Month)  
[3. Java and Unix I/O, section 4.7 of A Philosophy of Software Design - John Ousterhout](https://www.goodreads.com/en/book/show/39996759-a-philosophy-of-software-design)  
[4. A web of objects, chapter 2 of Growing Object-Oriented Software, Guided by Tests - Steve Freeman, Nat Pryce](https://www.goodreads.com/book/show/4268826-growing-object-oriented-software-guided-by-tests)
[5 Choosing Names, section 2.8 of 99 bottles of OOP - Sandy Metz](https://www.goodreads.com/book/show/31183020-99-bottles-of-oop)


# Does not repeat itself

# Does not contain superfluous parts


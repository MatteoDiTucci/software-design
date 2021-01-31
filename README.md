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
suggest you to go through the table of contents: you might find one or two things worth a quick read. 
As a last consideration, the book is heavily skewed towards object-oriented programming though many advices apply to all [programming paradigms](http://www.cs.albany.edu/~sdc/CSI500/Downloads/ProgrammingParadigmsVanRoyChapter.pdf).
Code examples are written in Kotlin, but they are very basic so you do not need any prior knowledge (Kotlin enthusiasts will forgive me some syntax choices :) ).

### Who I am
My name is Matteo Di Tucci, I currently work at ThoughtWorks.
I picked up coding in university because I did not know what to do when growing up.
I still do not know, but along the way I pleasantly discovered I like software and its design.

### Teach me back
I really appreciate any feedback about the book and my current understanding of software design.
You can share any feedback by creating a pull request.
If you do not know what a pull request is, take a look [here](https://guides.github.com/activities/hello-world/#:~:text=Pull%20Requests%20are%20the%20heart,merge%20them%20into%20their%20branch.&text=You%20can%20even%20open%20pull,repository%20and%20merge%20them%20yourself.).

### Simple code
It is hard to define what good design looks like: nobody was able to come up with a formal definition so far. 
The thing that came closer to that is the [C2 Wiki](https://wiki.c2.com/?XpSimplicityRules) simple code rules.  
Simple code:

* Passes all tests
* Expresses intent
* Does not repeat itself
* Does not contain superfluous parts

If a piece of code respects all those 4 rules then we can consider it good code.  
This book is divided in 4 sections, one for each the simple code rules.

# Tables of contents

### 1. Passes all tests
#### 1.1  [Test pyramid](#test-pyramid)
#### 1.2  [If testing is hard, inject what you need to verify](#if-testing-is-hard-inject-what-you-need-to-verify)
#### 1.3  Mock vs stub vs spy
#### 1.4  Test naming
#### 1.5  Test coverage is not enough: parameterised tests
#### 1.6  Tests must be reproducible
^ no Math.random() nor LocalDate.now()
#### 1.7  Do not test libraries
#### 1.8  Dry vs moist tests
#### 1.9  Test driven design (TDD)
^ Here mention sources to Outside-in vs inside-out TDD and Classical vs mockist TDD
#### 1.10  Component tests vs end-to-end tests vs monitoring tradeoffs
#### 1.11  Contract testing
#### 1.12  Do not use production constants
#### 1.13  Performance tests
#### 1.14  Linting
### 2. Expresses intent
#### 2.1 [Naming](#naming)
#### 2.2 [Deep and narrow classes](#deep-and-narrow-classes)
#### 2.3 Small classes and short methods
#### 2.4 SOLID principles
#### 2.5 Usually composition is better than inheritance
#### 2.6 Generalise edge cases
#### 2.7 Immutability
#### 2.8 Comments the why
#### 2.9 Visual indentation
^ Kevlin Henney talk
#### 2.10 Folder structure
#### 2.11 Encapsulation
### 3. Does not repeat itself
#### 3.1 [One single authoritative knowledge representation](#One-single-authoritative-knowledge-representation)
#### 3.2 [Do not abstract by visual pattern matching](#Do-not-abstract-by-visual-pattern-matching)
#### 3.3 Polymorphism
### 4. Does not contain superfluous parts
### 4.1 [You aren't going to need it](#You-aren't-going-to-need-it)
#### 4.2 [Define what is superfluous](#Define-what-is-superfluous)
#### 4.3 [Do not abuse design patterns](#Do-not-abuse-design-patterns)

# Passes all tests

### Test pyramid
There are different kinds of tests, each one meant for a different purpose. Before describing the most common ones, let's 
clarify in advance a couple of terms. For _service_ we mean a bunch of code deployed as a whole. For
_external dependencies_ we mean anything that is reached over the network, like a database, another team REST endpoint, a queue, third party apis, etc.   

* **Acceptance tests**  
  Check that a feature behaves as expected across all service layers (e.g. back end and front end) or even
  across different services. External dependencies are replaced through libraries like [LocalStack](https://github.com/localstack/localstack) 
  or [Wiremock](http://wiremock.org/). In a web application for instance, acceptance tests are defined with tools like [Selenium](https://www.selenium.dev/) 
  or [Cypress](https://www.cypress.io/).
* **Functional tests**  
  Check that a feature behaves as expected considering a single layer of a service. For example, if a service have both back end and 
  front end, there will be distinct functional tests for the back end and front end. External dependencies are replaced either 
  by libraries like [LocalStack](https://github.com/localstack/localstack) or by [doubles](#Mock-vs-stub-vs-spy). 
  Functional tests are also called component tests.
* **Integration tests**  
  Check that the service integrates correctly with external dependencies. External dependencies are replaced by libraries like
  [LocalStack](https://github.com/localstack/localstack) or [Wiremock](http://wiremock.org/). If you use code [doubles](#Mock-vs-stub-vs-spy)
  for the external dependencies then it is a unit test.
* **Unit tests**  
  Check that a class behaves as expected. Unit tests are most valuable when testing business logic: if a class is just a 
  [delegator](https://en.wikipedia.org/wiki/Delegation_pattern) or just coordinates other classes, do not use unit tests
  as functional tests already provide coverage. If the class under test uses other classes whose construction is cumbersome,
  those can be replaced with   [doubles](#Mock-vs-stub-vs-spy)  


The above list is ordered by how much time a test takes to execute, from the slowest (acceptance) to the fastest (unit). 
For this reason, it is recommended to have a pyramid of tests: a handful of acceptance tests, some functional and infrastructure
tests and many unit tests. In particular:

* **Acceptance tests**  
  Only for default uses of a feature
* **Functional tests**  
  For both default and exceptional-erroneous uses of a feature
* **Integration tests**  
  For both default and exceptional-erroneous integrations with external dependencies
* **Unit tests**  
  For both default and exceptional-erroneous usage of a single class
  

[1] [Test Pyramid (in short) - Martin Fowler](https://martinfowler.com/bliki/TestPyramid.html)  
[2] [Test Pyramid (in depth) - Ham Vocke](https://martinfowler.com/articles/practical-test-pyramid.html)  
[3] [Growing Object-Oriented Software, Guided by Tests - Steve Freeman, Nat Pryce](https://www.goodreads.com/book/show/4268826-growing-object-oriented-software-guided-by-tests)  
[4] ["Testing shows the presence, not the absence of bugs" - Edsger W. Dijkstra](https://blog.cleancoder.com/uncle-bob/2016/06/10/MutationTesting.html)   
[5] [Fixing a Test Hourglass, Google testing blog - Alan Myrvold](https://testing.googleblog.com/2020/11/fixing-test-hourglass.html)



### If testing is hard, inject what you need to verify
When you have a hard time testing something, the solution is usually to inject the thing you would like to verify.  
Suppose you have a car class storing passengers by their name.

```
class Car {
  private val passengers: MutableSet<String> = HashSet()
    
  fun storePassenger(name: String) {
    passengers.add(name)
  }
}
```

How to test that the method `storePassenger(name: String)` stores a name into the `passengers` set?
The typical solution is to define another method in `Car` to check if it contains a passenger.

```
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

```
@Test
fun 'stores the names of the passengers'() {
  val car = Car()
  
  car.storePassenger("Andrea")

  assertTrue(passengers.containsPassenger("Andrea"))
}
```

However, this is already a disappointment because we are forced to write a public method just for the sake of testing.  
Moreover, what if by specifications we must prevent any other code to query `Car` about its passengers? 
The solution is to inject the `passengers` set a construction time.

```
class Car(private val passengers: MutableSet<String>) {

  fun storePassenger(name: String) {
    passengers.add(name)
  }
}
```

Now we can test like follows

```
@Test
fun 'stores the names of the passengers'() {
  val passengers = HashSet()
  val car = Car(passengers)
  
  car.storePassenger("Andrea")

  assertTrue(passengers.contains("Andrea"))
}
```

Injecting the `passengers` set led to two benefits:
* No code is written just for testing purposes
* The class becomes independent of the data structure used to store the passengers, making the code more modular.

[1] [Context independence section, chapter 6 of Growing Object-Oriented Software, Guided by Tests - Steve Freeman, Nat Pryce](https://www.goodreads.com/book/show/4268826-growing-object-oriented-software-guided-by-tests)


### Mock vs stub vs spy

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
we can ask two questions. The first one is: if we remove the how from the `BiDimensionalCoordinates` names, does the class 
still express intent? Let's see.

```
class Coordinates(private val x: Int, private val y: Int)
  
  fun sum(addend: Coordinates): Coordinates {
    return Coordinates(x + addend.x, y + addend.y)
  }
}
```

The answer is yes, as the above class still expresses clear intent, but without bothering the reader with the noise of the how.
The second questions is: would the naming in `BiDimensionalCoordinates` still make sense if we were to switch to three-dimensional 
coordinates? Let's see.

```
class BiDimensionalCoordinates(private val x: Int, private val y: Int, private val z: Int) {
  
  fun sumByXandY(addend: BiDimensionalCoordinates): BiDimensionalCoordinates {
    return BiDimensionalCoordinates(x + addend.x, y + addend.y, z + addend.z)
  }
}
```

The answer is no. `BiDimensionalCoordinates` now does not express its intent anymore as it is lying to the reader: 
the names suggest two dimensions coordinates when it is actually three. However, the `Coordinates` class would still express 
its intent even when using three dimensions as shown below.

```
class Coordinates(private val x: Int, private val y: Int, private val z: Int) {
  
  fun sum(addend: Coordinates): Coordinates {
    return Coordinates(x + addend.x, y + addend.y, z + addend.z)
  }
}
```


[1] [Names, section 1.1.1 of 99 bottles of OOP - Sandy Metz](https://www.goodreads.com/book/show/31183020-99-bottles-of-oop)  
[2] [Choosing Names, section 2.8 of 99 bottles of OOP - Sandy Metz](https://www.goodreads.com/book/show/31183020-99-bottles-of-oop)  
[3] [Meaningful Names, chapter 2 of Clean Code - Robert C. Martin](https://www.goodreads.com/book/show/3735293-clean-code)  
[4] [Developing a Ubiquitous Language, chapter 2 of Domain-Driven Design Distilled - Vaughn Vernon](https://www.barnesandnoble.com/w/domain-driven-design-distilled-vaughn-vernon/1124175630)  


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

Now the `main` method is more readable. Complexity (the for-loop) has not disappeared, it just moved from `App` to `FizzBuzz`.  
However, such a shift becomes remarkable if applied to a codebase with many classes:
* In the first approach, the for-cycle is repeated every time a piece of code interacts with `FizzBuzz`.  
  In the second approach, we are guaranteed the for-cycle is written only once, inside `FizzBuzz`.
* If `main` interacted with 4 classes each one using the first approach of `FizzBuzz`, `main` would contain 4 for-loops.  
  If the 4 classes were to use the second approach, `main` will contain zero for-loops.

To summarise with a catchphrase, classes should be narrow and deep:
* narrow means few public methods with few input parameters
* deep means public methods handle as much complexity as possible for the caller

[1] ["Bad programmers worry about the code. Good programmers worry about data structures and their relationships" - Linus Torvalds](https://lwn.net/Articles/193245/)  
[2] ["Show me your tables, and I won't usually need your flowcharts; they'll be obvious.", chapter 9 of The Mythical Man-Month - Fred Brooks](https://www.goodreads.com/book/show/13629.The_Mythical_Man_Month)  
[3] [ Java and Unix I/O, section 4.7 of A Philosophy of Software Design - John Ousterhout](https://www.goodreads.com/en/book/show/39996759-a-philosophy-of-software-design)  
[4] [ A web of objects, chapter 2 of Growing Object-Oriented Software, Guided by Tests - Steve Freeman, Nat Pryce](https://www.goodreads.com/book/show/4268826-growing-object-oriented-software-guided-by-tests)  
[5] [Choosing Names, section 2.8 of 99 bottles of OOP - Sandy Metz](https://www.goodreads.com/book/show/31183020-99-bottles-of-oop)


# Does not repeat itself

### One single authoritative knowledge representation
Duplicated code is harmful because it makes changes expensive. Worse than that, it conceals the code intent.
Let's take a look at the following code.

```
class Person(private val name: String) {

  fun name(): String {
    return name
  }
}

class Job(private val name:String) {

  fun name(): String {
    return name
  }
}

class App {

  fun main() {
    val person = Person("Andrea")
    val job = Job("developer")

    val result = person.name() + " is a " + job.name() //Andrea is a developer
  }
}
```

Now let's assume we want to have an ellipsis when `Person` or `Job` names are longer than 5 characters. One solution
could be to modify the `name` method of both `Person` and `Job` like follows.

```
class Person(private val name: String) {

  fun name(): String {
    if (name.length > 5) {
      return name.substring(0, 5) + "..."
    }
    return name
  }
}

class Job(private val name:String) {

  fun name(): String {
    if (name.length > 5) {
      return name.substring(0, 5) + "..."
    }
    return name
  }
}

class App {

  fun main() {
    val person = Person("Andrea")
    val job = Job("developer")

    val result = person.name() + " is a " + job.name() //Andre... is a devel...
  }
}
```


In the above code, changing the ellipsis threshold from 5 to 10 characters will affect the `name` method of both `Person`and `Job` classes.
This sounds trivial, but in a large codebase where duplication is widespread even small changes like this are expensive. 
Moreover, there is a risk to introduce bugs as changes might not be replicated across all classes by mistake. However, if duplication 
is removed like in the following code, we are guaranteed all ellipsis related changes will always affect only one class.

```
class Person(private val name: String) {

  fun name(): String {
    return name
  }
}

class Job(private val name:String) {

  fun name(): String {
    return name
  }
}

class Formatter {

  fun format(name: String): String {
    if (name.length > 5) {
      return name.substring(0, 5) + "..."
    }
    return name
  }
}

class App {

  fun main() {
    val person = Person("Andrea")
    val job = Job("developer")
    val formatter = Formatter()

    val formattedPerson = formatter.format(person.name())
    val formattedJob = formatter.format(job.name())
    
    val result = formattedPerson + " is a " + formattedJob //Andre... is a devel...
  }
}
```

The above code has another subtle benefit, more important than reducing the amount of work for future changes. It is now clear which class
is responsible for the formatting knowledge like ellipsis. In this way the code intent is clearer, making it easier to reason about it.

[1] [DRY, The evils of duplication, topic 9 of The Pragmatic Programmer - David Thomas, Andrew Hunt](https://www.goodreads.com/book/show/4099.The_Pragmatic_Programmer)  
[2] [Once and only once, Extreme Programming Explained - Kent Beck](https://www.goodreads.com/book/show/67833.Extreme_Programming_Explained)  
[3] [Don't Repeat Yourself, chapter 30 of 97 Things Every Programmer Should Know - Kevlin Henney](https://www.goodreads.com/book/show/7003902-97-things-every-programmer-should-know)  

### Do not abstract by visual pattern matching
Two pieces of code that are identical but represent two distinct concepts should stay separated: they are not duplication. 
Let's consider the example from the previous section where both `Person` and `Job` classes introduced an ellipsis when their
name was longer than 5 characters.

```
class Person(private val name: String) {

  fun name(): String {
    if (name.length > 5) {
      return name.substring(0, 5) + "..."
    }
    return name
  }
}

class Job(private val name:String) {

  fun name(): String {
    if (name.length > 5) {
      return name.substring(0, 5) + "..."
    }
    return name
  }
}

class App {

  fun main() {
    val person = Person("Andrea")
    val job = Job("developer")

    val result = person.name() + " is a " + job.name() //Andre... is a devel...
  }
}
```

When seeing a piece of like the above the immediate instinct is to remove duplication as was done in the previous section.

```
class Person(private val name: String) {

  fun name(): String {
    return name
  }
}

class Job(private val name:String) {

  fun name(): String {
    return name
  }
}

class Formatter {

  fun format(name: String): String {
    if (name.length > 5) {
      return name.substring(0, 5) + "..."
    }
    return name
  }
}

class App {

  fun main() {
    val person = Person("Andrea")
    val job = Job("developer")
    val formatter = Formatter()

    val formattedPerson = formatter.format(person.name())
    val formattedJob = formatter.format(job.name())
    
    val result = formattedPerson + " is a " + formattedJob //Andre... is a devel...
  }
}
```

However, let's now assume `Job` needs the ellipsis after 3 characters instead of 5. How does the code change?
`Formatter` needs to know if the name is coming from `Job` or `Person`to apply the ellipsis at 3 or 5 characters respectively.
For instance, this piece of information can be passed as an input parameter of the `format` method like follows.

```
class Person(private val name: String) {

  fun name(): String {
    return name
  }
}

class Job(private val name:String) {

  fun name(): String {
    return name
  }
}

class Formatter {

  fun format(name: String, isPerson: Boolean): String {
    if (isPerson) {
      return formatPersonName(name)
    }

    return formatJobName(name)
  }

  private fun formatJobName(name: String): String {
    if (name.length > 3) {
      return name.substring(0, 3) + "..."
    }
    return name
  }

  private fun formatPersonName(name: String): String {
    if (name.length > 5) {
      return name.substring(0, 5) + "..."
    }
    return name
  }
}

class App {

  fun main() {
    val person = Person("Andrea")
    val job = Job("developer")
    val formatter = Formatter()

    val formattedPerson = formatter.format(person.name(), true)
    val formattedJob = formatter.format(job.name(), false)

    val result = formattedPerson + " is a " + formattedJob
  }
}
```

In the above code, `Formatter` has a lot of if-else logic making it harder to understand. This complexity stems from 
not foreseeing that the ellipsis rules for `Person` and `Job` were going to be different, even if they looked initially
the same. In real life this kind of scenario can be hard to anticipate, especially if specifications are not entirely known.
In doubt, the rule of thumb is to remove duplication only after it occurs more than two times.


[1] ["duplication is far cheaper than the wrong abstraction", RailsConf 2014, all the little things talk - Sandy Metz](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction)  
[1] [Rule of three, When we should Refactor? chapter 2 of Refactoring - Martin Fowler](https://www.goodreads.com/en/book/show/44936.Refactoring)  

# Does not contain superfluous parts

### You aren't going to need it

[1] [Refactoring, Architecture, and YAGNI, chapter 2 of Refactoring - Martin Fowler](https://www.goodreads.com/en/book/show/44936.Refactoring)  
[2] [Chapter 17 of Extreme Programming Explained - Kent Beck](https://www.goodreads.com/book/show/67833.Extreme_Programming_Explained)  

### Define what is superfluous 
Error handling (division by zero), performance.

### Do not abuse design patterns
Design patterns are great to codify recurring design problems and their typical solutions. However, design patterns by themselves 
do not guarantee simple code, and actually they can end up pushing in the opposite direction. They are just a tool that 
has to be adapted to the context where they are used, not applied to the letter.

[1] [Head First Design Patterns - Eric Freeman, Elisabeth Robson](https://www.goodreads.com/book/show/58128.Head_First_Design_Patterns)

Introduction
Why read this book
Software design is a craft not a science. To master it you cannot just study a book that explains comprehensively all the theory behind it. You have to read the literature of many great developers, looking for advice that is repeated: that's what is currently considered good software design. However, the literature is vast and it takes a great deal of time to get the big picture out of single sources. This book makes three promises:
to be a catalogue of everything that contributes to good software design
to be very succinct
to provide trustworthy sources to go deeper
Who should read this book
The book is intended mainly for people who just started their journey into software design or who are in the middle of it. If you have being seriously studying the topic for 5+ years, it is likely you are not going to find big surprises. However, I still suggest that you go through the table of contents: you might find one or two things worth a quick read. As a last consideration, the book is heavily skewed towards object-oriented programming though much of the advice applies to all programming paradigms. Code examples are written in Kotlin, but they are very basic so you do not need any prior knowledge (Kotlin enthusiasts will forgive me some syntax choices :) ).
Who I am
My name is Matteo Di Tucci, I currently work at ThoughtWorks. I picked up coding in university because I did not know what to do when growing up. I still do not know, but along the way I pleasantly discovered I like software and its design.
Teach me back
I really appreciate any feedback about the book and my current understanding of software design. You can share any feedback by creating a pull request. If you do not know what a pull request is, take a look here.
Simple code
It is hard to define what good design looks like: nobody has been able to come up with a formal definition so far. The thing that has come closist are the C2 Wiki simple code rules.
Simple code:
Passes all tests
Expresses intent
Does not repeat itself
Does not contain superfluous parts
If a piece of code respects all those 4 rules then we can consider it good code.
This book is divided in 4 sections, one for each the simple code rules.
Tables of contents
1. Passes all tests
   1.1 Test pyramid
   1.2 If testing is hard, inject what you need to verify
   1.3 Mock vs stub vs spy
   1.4 Test naming
   1.5 Test coverage is not enough: parameterised tests
   1.6 Tests must be reproducible
   ^ no Math.random() nor LocalDate.now()
   1.7 Do not test libraries
   1.8 Dry vs moist tests
   1.9 Test driven design (TDD)
   ^ Here mention sources to Outside-in vs inside-out TDD and Classical vs mockist TDD
   1.10 Component tests vs end-to-end tests vs monitoring tradeoffs
   1.11 Contract testing
   1.12 Do not use production constants
   1.13 Performance tests
   1.14 Linting
   1.15 How to test UI (screenshot and snapshot testing)
2. Expresses intent
   2.1 Naming
   2.2 Deep and narrow classes
   2.3 Types
   2.4 Small classes and short methods
   2.5 SOLID principles
   2.6 Usually composition is better than inheritance
   2.7 Generalise edge cases
   2.8 Immutability
   2.9 Comments the why
   2.10 Visual indentation
   ^ Kevlin Henney talk
   2.10 Folder structure
   2.11 Encapsulation
3. Does not repeat itself
   3.1 One single authoritative knowledge representation
   3.2 Do not abstract by visual pattern matching
   3.3 Polymorphism
4. Does not contain superfluous parts
   4.1 You ain't going to need it
   4.2 Clarify what is superfluous
   4.3 Do not abuse design patterns
   Passes all tests
   Test pyramid
   There are different kinds of tests, each one meant for a different purpose. Before describing the most common ones, let's clarify in advance a couple of terms. By service we mean a bunch of code deployed as a whole. By external dependencies we mean anything that is reached over the network, like a database, another team's REST endpoint, a queue, third party apis, etc.
   Acceptance tests Could mention that these are also sometimes referred to as end2end?
   Check that a feature behaves as expected across all service layers (e.g. back end and front end) or even across different services. External dependencies are replaced through libraries like LocalStack or Wiremock. In a web application for instance, acceptance tests are defined with tools like Selenium or Cypress.
   Functional tests
   Check that a feature behaves as expected considering a single layer of a service. For example, if a service have both back end and front end, there will be distinct functional tests for the back end and front end. External dependencies are replaced either by libraries like LocalStack or by doubles. Functional tests are also called component tests.
   Integration tests
   Check that a service integrates correctly with external dependencies. External dependencies are replaced by libraries like LocalStack or Wiremock. If you use code doubles for the external dependencies then it is a unit test.
   Unit tests Here could the concept of "The smallest unit of value/work be mentioned?" I think Kent Beck has a quote somewhere Check that code inside one class behaves as expected. Unit tests are most valuable when testing business logic: if a class is just a delegator or just coordinates other classes, do not use unit tests as functional tests already provide coverage. If the class under test uses other classes whose construction is cumbersome, those can be replaced with doubles
   The above list is ordered by how much time a test takes to execute, from the slowest (acceptance) to the fastest (unit). For this reason, it is recommended to have a pyramid of tests: a handful of acceptance tests, some functional and integration tests and many unit tests. In particular:
   Acceptance tests
   Only for default uses of a feature
   Functional tests
   For both default and exceptional-erroneous uses of a feature
   Integration tests
   For both default and exceptional-erroneous integrations with external dependencies
   Unit tests
   For both default and exceptional-erroneous usage of a single class
   [1] Test Pyramid (in short) - Martin Fowler
   [2] Test Pyramid (in depth) - Ham Vocke
   [3] Growing Object-Oriented Software, Guided by Tests - Steve Freeman, Nat Pryce
   [4] "Testing shows the presence, not the absence of bugs" - Edsger W. Dijkstra
   [5] Fixing a Test Hourglass, Google testing blog - Alan Myrvold [6] Characterization tests, chapter 13 of Working Effectively with Legacy Code - Michael Feathers
   If testing is hard, inject what you need to verify
   When you have a hard time testing something, the solution is usually to inject the thing you would like to verify.
   Suppose you have a Car class storing passengers by their name.
   class Car {
   private val passengers: MutableSet<String> = HashSet()

fun storePassenger(name: String) {
passengers.add(name)
}
}
How could you test that the method storePassenger(name: String) stores a name into the passengers set? The typical solution is to define another method in Car to check if it contains a passenger.
class Car {
private val passengers: MutableSet<String> = HashSet()

fun storePassenger(name: String) {
passengers.add(name)
}

fun containsPassenger(name: String): Boolean {
return passengers.contains(name)
}
}
So we can write the following test
@Test
fun `stores the names of the passengers`() {
val car = Car()

car.storePassenger("Andrea")

assertTrue(passengers.containsPassenger("Andrea"))
}
However, this is already a disappointment because we are forced to write a public method just for the sake of testing. Moreover, what if by specifications we must prevent any other code to query Car about its passengers? The solution is to inject the passengers set a construction time.
class Car(private val passengers: MutableSet<String>) {

fun storePassenger(name: String) {
passengers.add(name)
}
}
Now we can test like follows.
@Test
fun `stores the names of the passengers`() {
val passengers = HashSet()
val car = Car(passengers)

car.storePassenger("Andrea")

assertTrue(passengers.contains("Andrea"))
}
Injecting the passengers set, we can get rid of the useless containsPassenger method. Furthermore, we have now the opportunity to make our code more modular, making Car independent of the data structure used to store the passengers. For this, we can use MutableCollection<String> instead of MutableSet<String>
class Car(private val passengers: MutableCollection<String>) {

fun storePassenger(name: String) {
passengers.add(name)
}
}
[1] Context independence section, chapter 6 of Growing Object-Oriented Software, Guided by Tests - Steve Freeman, Nat Pryce
Mock vs stub vs spy
:( I was looking forward to this part. I came accross this issue when I was doing my bootcamp, and had real issues finding a clear and concise definition that highlighted the difference between a mock and a stub :)
Expresses intent
Naming
Great naming lets/allows us to understand what the code means without inspecting its details. In fact, we should be able to understand an entire codebase just reading the names of classes, methods and variables, never looking at what they contain. This clarity is reached by naming classes, methods and variables after what they do instead of how they do it. The how can be looked up later if more details are needed.
The benefits are two:
names do not contain any noise
if implementation changes, the name remains relevant
Let's consider the following class
class BiDimensionalCoordinates(private val x: Int, private val y: Int) {

fun sumByXandY(addend: BiDimensionalCoordinates): BiDimensionalCoordinates {
return BiDimensionalCoordinates(x + addend.x, y + addend.y)
}
}
In the above code, both class and method names leak the how. Given the distinction we made about the what and the how, we can ask two questions. The first one is: if we remove the how from the BiDimensionalCoordinates names, does the class still express intent? Let's see.
class Coordinates(private val x: Int, private val y: Int) {

fun sum(addend: Coordinates): Coordinates {
return Coordinates(x + addend.x, y + addend.y)
}
}
The answer is yes, as the above class still expresses clear intent, without even bothering the reader with the noise of the how. The second questions is: would the naming in BiDimensionalCoordinates still make sense if we were to switch to three-dimensional coordinates? Let's see.
class BiDimensionalCoordinates(private val x: Int, private val y: Int, private val z: Int) {

fun sumByXandY(addend: BiDimensionalCoordinates): BiDimensionalCoordinates {
return BiDimensionalCoordinates(x + addend.x, y + addend.y, z + addend.z)
}
}
The answer is no. BiDimensionalCoordinates now does not express its intent anymore as it is lying to the reader: the names suggest two dimensions coordinates when they are actually three. However, the Coordinates class would still express its intent even when using three dimensions as shown below.
class Coordinates(private val x: Int, private val y: Int, private val z: Int) {

fun sum(addend: Coordinates): Coordinates {
return Coordinates(x + addend.x, y + addend.y, z + addend.z)
}
}
[1] Names, section 1.1.1 of 99 bottles of OOP - Sandy Metz
[2] Choosing Names, section 2.8 of 99 bottles of OOP - Sandy Metz
[3] Meaningful Names, chapter 2 of Clean Code - Robert C. Martin
[4] Developing a Ubiquitous Language, chapter 2 of Domain-Driven Design Distilled - Vaughn Vernon suggest naming book [5] The Art of Readable Code
Deep and narrow classes
We are doomed to write convoluted code when interacting with classes that have poor public methods. On the contrary, we are led into writing readable code when interacting with classes that have great public methods.
Let's consider the following play public method for the FizzBuzz game.
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
The above signature of the play method leads to the following code when playing with the numbers between 1 and 100.
class App {

fun main() {

    val fizzBuzz = FizzBuzz()    
    val result = ArrayList<String>()
    
    for (number in 1..100)
      result.add(fizzBuzz.play(number))
}
}
Now, let's change the signature of the play method like follows.
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
The above signature of the play method leads to the following code when playing with the numbers between 1 and 100.
class App {

fun main() {

    val fizzBuzz = FizzBuzz()
    val result = fizzBuzz.play(1, 100)        
}
}
Now the main method is more readable. The for-loop has not disappeared, it just moved from App to FizzBuzz.
However, such a shift becomes remarkable if applied to a codebase with many classes:
In the first approach, the for-cycle is repeated every time a piece of code interacts with FizzBuzz.
In the second approach, we are guaranteed the for-cycle is written only once: inside FizzBuzz.
If main interacted with 4 classes each one using the first approach of FizzBuzz, main would contain 4 for-loops.
If the 4 classes were to use the second approach, main will contain zero for-loops.
To summarise with a catchphrase, classes should be narrow and deep:
narrow means few public methods with few input parameters
deep means public methods get a lot of things done for the caller
[1] "Bad programmers worry about the code. Good programmers worry about data structures and their relationships" - Linus Torvalds
[2] "Show me your tables, and I won't usually need your flowcharts; they'll be obvious.", chapter 9 of The Mythical Man-Month - Fred Brooks
[3] Java and Unix I/O, section 4.7 of A Philosophy of Software Design - John Ousterhout
[4] A web of objects, chapter 2 of Growing Object-Oriented Software, Guided by Tests - Steve Freeman, Nat Pryce
[5] Choosing Names, section 2.8 of 99 bottles of OOP - Sandy Metz
Does not repeat itself
One single authoritative knowledge representation
Duplicated knowledge is harmful because it makes changes expensive. Worse than that, it conceals the code intent. Let's take a look at the following code.
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
Now let's assume we want to have an ellipsis when Person or Job names are longer than 5 characters. One solution could be to modify the name method of both Person and Job like follows.
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
In the above code, changing the ellipsis threshold from 5 to 10 characters will affect the name method of both Personand Job classes. Moreover, there is a risk of introducing bugs as changes might not be replicated across both classes by mistake. The problem is that the concept of name is spread across Person and Job instead if being a standalone class like follows.
class Name(private val name: String) {

fun format(): String {
if (name.length > 5) {
return name.substring(0, 5) + "..."
}
return name
}
}

class Person(private val name: Name) {

fun format(): String {
return name.format()
}
}

class Job(private val name:Name) {

fun format(): String {
return name.format()
}
}

class App {

fun main() {
val person = Person(Name("Andrea"))
val job = Job(Name("developer"))

    val result = person.format() + " is a " + job.format() //Andre... is a devel...
}
}
Now the intent is clearer, making it easier to reason about the code.
As a final note, be mindful that knowledge duplication might be there even if the code looks different. The typical example is object-relational mapping libraries and SQL initialisation scripts: who of the two represents the knowledge of database tables creation?
[1] DRY, The evils of duplication, topic 9 of The Pragmatic Programmer - David Thomas, Andrew Hunt
[2] Once and only once, Extreme Programming Explained - Kent Beck
[3] Don't Repeat Yourself, chapter 30 of 97 Things Every Programmer Should Know - Kevlin Henney
Do not abstract by visual pattern matching
Two pieces of code that are identical but represent two distinct concepts should stay separated: they are not duplication. Let's consider the example where we are asked to calculate the perimeter of rectangles and rhombuses.
class Rhombus(private val width: Int, private val height: Int) {

fun perimeter(): Int {
return 2 * (width + height)
}
}

class Rectangle(private val width: Int, private val height: Int) {

fun perimeter(): Int {
return 2 * (width + height)
}
}
When seeing a piece of code like the above, one might be tempted to remove the duplication of the perimeter method like follows.
class Shape(private val width: Int, private val height: Int) {

fun perimeter(): Int {
return 2 * (width + height)
}
}
However, what happens if after some time we are asked to calculate the perimeter of a circle? Now the concept represented by Shape does not relate to circle, which is definitely a shape though. There are many ways to refactor this situation, for instance making shape an interface, but in a large codebase these refactorings are more expensive than leaving duplicated code until we understand it is duplicated knowledge. It is hard to build the right knowledge model, especially if specifications are not entirely known or the domain is complex. When in doubt, the rule of thumb is to remove duplication only after it occurs more than two times.
[1] "duplication is far cheaper than the wrong abstraction", RailsConf 2014, all the little things talk - Sandy Metz
[1] Rule of three, When we should Refactor? chapter 2 ofa Refactoring - Martin Fowler
Does not contain superfluous parts
You ain't going to need it (Also referred to as AGNI?)
Writing code that is beyond specifications is not only a waste of time, but it also introduces unnecessary complexity. This complexity consists mainly in more code and tests to understand and maintain. Worse than that, superfluous code makes it harder to change code in the future. Let's consider an example given the following specifications:
Create a pedestrian traffic light allowing to walk if green, denying it if red
The traffic light can change color from red to green and vice versa
class TrafficLight(private var colour: TrafficLightColour) {

fun changeColour() {
would this not need a return statement in Kotlin? Also, the assignation is redundant?
colour = colour.changeColour()
}

fun possibleToWalk(): Boolean {
return colour.possibleToWalk()
}
}

enum class TrafficLightColour {
RED {
override fun changeColour() = GREEN
override fun possibleToWalk() = false
},
GREEN {
override fun changeColour() = RED
override fun possibleToWalk() = true
};

abstract fun changeColour(): TrafficLightColour
abstract fun possibleToWalk(): Boolean
}
The above code satisfies the specifications. However, one might say that traffic lights for pedestrians are usually timed: you press a button and it will take some seconds before the colour changes. This was not requested, but as it seems to be a more faithful representation of a pedestrian traffic light, so one might decide to go ahead like follows.
class TrafficLight(private var colour: TrafficLightColour) {

fun changeColour(milliSecondsToWait: Long) {
waitFor(milliSecondsToWait)
see above colour = colour.changeColour(colour)
}

fun possibleToWalk(): Boolean {
return colour.possibleToWalk()
}

private fun waitFor(milliSecondsToWait: Long) {
Thread.sleep(milliSecondsToWait)
}
}

enum class TrafficLightColour {
RED {
override fun changeColour() = GREEN
override fun possibleToWalk() = false
},
GREEN {
override fun changeColour() = RED
override fun possibleToWalk() = true
};

abstract fun changeColour(): TrafficLightColour
abstract fun possibleToWalk(): Boolean
}
Changing the behaviour of changeColour had no benefits for what we are concerned with the current specifications. However, it added useless code inside the TrafficLight class and it forced all callers of changeColour to provide a value for milliSecondsToWait which, given the current spec, will always be 0.
*** This is a really nice example which made me think about some of the rubbish I've written in the past***
[1] Refactoring, Architecture, and YAGNI, chapter 2 of Refactoring - Martin Fowler
[2] Chapter 17 of Extreme Programming Explained - Kent Beck
[3] Incidental vs accidental complexity, No Silver Bullet paper - Fred Brooks
Clarify what is superfluous
Sometimes the you aren't going to need it principle is abused, leading to code that does not pass all tests. This usually happens in the form of assuming specifications. Let's consider the following example where we are required to build a piece of code that:
stores fruits by their names
does not store a fruit name more than once
given a fruit name, returns true if it has been previously stored
class FruitInventory {
private val fruits = ArrayList<String>()

fun storeFruit(name: String) {
if (contains(name))
return

    fruits.add(name)
}

fun contains(name: String): Boolean {
for(fruit in fruits) {
if (name == fruit)
return true
}
return false
}
}
The above code works just fine given the specifications. However, we might argue that fruits implemented as a list is not very performant and that we could make both contains and storeFruit faster by simply using a set. It is a mistake to dismiss such thought appealing to the you aren't going to need it principle. In fact, we would be assuming a specification about performance that is currently unknown. We should instead clarify the expected performance of FruitInventory: if it is irrelevant then the code is as good as it is, otherwise we need to change it as follows.
class FruitInventory {
private val fruits = HashSet<String>()

fun storeFruit(name: String) {
fruits.add(name)
}

fun contains(name: String): Boolean {
return fruits.contains(name)
}
}
Performance specifications aside, the above code using HashSet is superior as it better expresses intent. We could even argue that HashSet approach satisfies the specifications just fine and we aren't going to need all the for-loop complexity introduced by the ArrayList approach.
*** I like this example (HashSet versus ArrayList), not least because I have learnt something myself. I wonder however if the beginner will truly appreciate the difference, given the nuiance between one data structure and another? I'm trying to think of an example that would be succint like the one above (I have another about the time I implemented a search endpoint using elasticSearch, where I then had to dockerize the clusters and get the containers to talk to the those of the original application, none of which I had done before, all for a three day code challenge. I could have just used a HashSet like the one about facepalm. I did not get the position)***
[1] The full quote from Donald Knuth was: "We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil. Yet we should not pass up our opportunities in that critical 3%."
Do not abuse design patterns
Design patterns are great to catalogue recurring design problems and their typical solutions. However, design patterns by themselves do not guarantee simple code, and actually they can end up pushing in the opposite direction. They are just a tool that has to be adapted to the context where they are used, not applied to the letter.
[1] Head First Design Patterns - Eric Freeman, Elisabeth Robson


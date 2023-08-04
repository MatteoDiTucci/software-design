## Usually composition is better than inheritance
Inheritance and composition are both useful to share behaviour or state from one class to another.
However, inheritance comes with a bigger burden than composition. Let's clarify this with the following example.

```kotlin
open class Person(private val name: String) {
    fun name(): String {
        return "My name is $name"
    }
}

class Teacher(name: String) : Person(name)
class Student(name: String) : Person(name)

fun main() {
   println(Teacher("Lidia").name())   // My name is Lidia 
   println(Student("Cosimo").name())  // My name is Cosimo
}

```

So far so good, inheritance is making the method `name` reusable instead of having it duplicated in `Teacher` and `Student`.
However, let's assume we want to specify a minimum hourly wage for each Person:

```kotlin
import java.math.BigDecimal

open class Person(private val name: String) {
    fun name(): String {
        return "My name is $name"
    }
    open fun minHourlyWageInEur(): BigDecimal {
        return BigDecimal(10)
    }
}

class Teacher(name: String) : Person(name) 

class Student(name: String) : Person(name) {
    override fun minHourlyWageInEur(): BigDecimal {
        throw Exception("Students do not have a wage")
    }
}

fun main() {
    println(Teacher("Lidia").minHourlyWageInEur())  // 10
    println(Student("Cosimo").minHourlyWageInEur()) // throws an exception
}
```

You can see above that the method `minHourlyWageInEur` is a problem for `Student` as it is not supposed to have a wage.
That's why we are forced to override it to throw an Exception in case it is invoked by mistake.
Ideally, we would like to avoid for `Student` to implement `minHourlyWageInEur`, but because of inheritance, we are stuck with it.

Let's see instead how things would look like if we were to use composition:
```kotlin
import java.math.BigDecimal

class Person(private val name: String) {
    fun name(): String {
        return "My name is $name"
    }
    fun minHourlyWageInEur(): BigDecimal {
        return BigDecimal(10)
    }
}

class Teacher(private val person: Person) {
    fun name(): String {
        return "My name is $person.name()"
    }
    fun minHourlyWageInEur(): BigDecimal {
        return person.minHourlyWageInEur()
    }
}

class Student(private val person: Person) {
    fun name(): String {
        return person.name()
    }
}

fun main() {
    println(Teacher(Person("Lidia")).minHourlyWageInEur())  // 10
    println(Student(Person("Cosimo")).name())               // My name is Cosimo
}
```

We are still reusing the method `name` defined in `Person`, but we do not have the burden to support all the public methods 
of `Person`, among which the problematic `minHourlyWageInEur` for `Student`.

Inheritance is especially problematic if we extend classes that we do not control. For instance, when they are maintained 
by a different team or even worse when they are part of an external library. We will be forced to handle any future change on the
public methods of the class we inherited from.


Be mindful that inheritance is not intrinsically bad. Like any other tool it is just the way we use it which can be troublesome.
Despite inheritance usually being not great to share state or behaviour across classes, it is very useful to establish [is-a](https://en.wikipedia.org/wiki/Is-a) 
relationships which is one way to leverage [polymorphism](../does-not-repeat-itself/polymorphism.html)


#### Recommended reads
* [Decorating Objects, chapter 3 of Head First Design Patterns - Eric Freeman, Elizabeth Robson](https://www.goodreads.com/book/show/56083609-head-first-design-patterns)
* ["Duplication is far cheaper than the wrong abstraction", RailsConf 2014, all the little things talk - Sandy Metz](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction)

#### Teach me back
I really appreciate any [feedback](../introduction/introduction.html#teach-me-back) about the book and my current understanding of software design.
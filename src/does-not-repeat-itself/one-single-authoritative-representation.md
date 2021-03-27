## One single authoritative knowledge representation
Duplicated knowledge is harmful because it makes changes expensive. Worse than that, it conceals the code intent.
Let's take a look at the following code.

```kotlin
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

```kotlin
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
Moreover, there is a risk to introduce bugs as changes might not be replicated across both classes by mistake. The problem
is that the concept of name is spread across `Person` and `Job` instead if being a standalone class like follows.

```kotlin
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
```

Now the intent is clearer, making it easier to reason about the code.

As a final note, be mindful that knowledge duplication might be there even if the code looks different. The typical example
is [object-relational mapping libraries](https://en.wikipedia.org/wiki/Object%E2%80%93relational_mapping) and SQL
initialisation scripts: who of the two represents the knowledge of database tables creation?

<br/>  

#### Recommended reads
* [DRY, The evils of duplication, chapter 2 of The Pragmatic Programmer - David Thomas, Andrew Hunt](https://www.goodreads.com/book/show/4099.The_Pragmatic_Programmer)  
* [Once and only once, Extreme Programming Explained - Kent Beck](https://www.goodreads.com/book/show/67833.Extreme_Programming_Explained)  
* [Don't Repeat Yourself, chapter 30 of 97 Things Every Programmer Should Know - Kevlin Henney](https://www.goodreads.com/book/show/7003902-97-things-every-programmer-should-know)

<br/>  

[Teach me back ❤️](/introduction/introduction.html#teach-me-back)
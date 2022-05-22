## Wrap external libraries into custom classes

[Information hiding️](does-not-repeat-itself/information-hiding.md) also applies to external libraries. In fact,
swapping a library for another can unexpectedly
trigger many undesired changes across our codebase.

Let's take a look at the following code.

```kotlin
class MessageController(private val metrics: ExternalMetricLibrary) {

    fun handleMessage(message: String) {
        metrics.incrementMetric("total-messages-received")
    }
}

class UserController(private val metrics: ExternalMetricLibrary) {

    fun handleLogin(username: String) {
        metrics.incrementMetric("total-users-logged-in")
    }
}
```

In the above code, `ExternalMetricLibrary` is an external library that we use for monitoring purposes. 
The method `incrementMetric` takes in input the name of the metric to be incremented by one.
Let's assume we want to replace `ExternalMetricLibrary` with another library called `FasterExternalMetricLibrary` because it has better performance. If that's the case, we would be forced to replace all the occurrences of `ExternalMetricLibrary`, which in the above example are just two, but in a real life codebase can be many. 

To prevent this we can wrap `ExternalMetricLibrary` in a custom class so that it does not spread across the codebase.

```kotlin
class MessageController(private val metrics: Metrics) {

    fun handleMessage(message: String) {
        metrics.incrementMetric("total-messages-received")
    }
}

class UserController(private val metrics: Metrics) {

    fun handleLogin(username: String) {
        metrics.incrementMetric("total-users-logged-in")
    }
}

class Metrics(private val externalMetricLibrary: ExternalMetricLibrary) {
    fun incrementMetric(metricName: String) {
        this.externalMetricLibrary.incrementMetricOf(metricName)
    }
}
```

Now our class `Metrics` will be the only place in the codebase that knows about `ExternalMetricLibrary`. Whenever we
need to replace `ExternalMetricLibrary` in favour of `FasterExternalMetricLibrary`, only `Metrics` will be affected.
This ensures our codebase remains stable despite any library change we might have in the future. 

As a final note, wrapping external libraries into custom classes will also simplify testing as explained in the
[do not test external libraries](passes-all-tests/do-not-test-external-libraries.md) chapter.

<br/>

#### Recommended reads
* [Adapter pattern chapter of Head First Design Patterns - Eric Freeman, Elisabeth Robson](https://www.goodreads.com/book/show/58128.Head_First_Design_Patterns)

#### Teach me back

I really appreciate any [feedback](../introduction/introduction.html#teach-me-back) about the book and my current
understanding of software design.

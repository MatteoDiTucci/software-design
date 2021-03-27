## You aren't going to need it
Writing code that is beyond specifications is not only a waste of time, but it also introduces unnecessary complexity.
This complexity consists mainly in more code and tests to understand and maintain. Worse than that, superfluous code
makes it harder to change code in the future. Let's consider an example given the following specifications:
* Create a pedestrian traffic light allowing to walk if green, denying it if red
* The traffic light can change color from red to green and vice versa

```kotlin
class TrafficLight(private var colour: TrafficLightColour) {

  fun changeColour() {
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
```

The above code satisfies the specifications. However, one might say that traffic lights for pedestrians are usually timed: you press
a button and it will take some seconds before the colour changes. This was not requested, but as it seems to be a more
faithful representation of a pedestrian traffic light, so one might decide to go ahead like follows.

```kotlin
class TrafficLight(private var colour: TrafficLightColour) {

  fun changeColour(milliSecondsToWait: Long) {
    waitFor(milliSecondsToWait)
    colour = colour.changeColour(colour)
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
```

Changing the behaviour of `changeColour` had no benefits for what we are concerned with the current specifications. However,
it added useless code inside the `TrafficLight` class and it forced all callers of `changeColour` to provide a value for
`milliSecondsToWait` which, given the current spec, will always be 0.

<br/>  

#### Recommended reads
* [Refactoring, Architecture, and YAGNI, chapter 2 of Refactoring - Martin Fowler](https://www.goodreads.com/en/book/show/44936.Refactoring)    
* [Chapter 17 of Extreme Programming Explained - Kent Beck](https://www.goodreads.com/book/show/67833.Extreme_Programming_Explained)  
* [Incidental vs accidental complexity, No Silver Bullet paper - Fred Brooks](https://en.wikipedia.org/wiki/No_Silver_Bullet)

#### Teach me back
I really appreciate any [feedback]((/introduction/introduction.html#teach-me-back)) about the book and my current understanding of software design.
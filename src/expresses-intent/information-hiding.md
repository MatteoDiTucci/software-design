## Information Hiding
When objects expose their internal implementation, modifying code becomes harder. In fact, a single change can unexpectedly 
trigger many others across the code base. Let's take a look at the following example.

```kotlin
class Die {
  var value: Int = 1

  fun roll() {
    value = (1..6).random()
  }
}

class Game{

  fun play() {
    val player1Die = Die()
    val player2Die = Die()

    player1Die.roll()
    player2Die.roll()

    if(player1Die.value > player2Die.value) {
      print("Player 1 wins")
      return
    }
    if(player1Die.value < player2Die.value) {
      print("Player 2 wins")
      return
    }
    print("Draw")
  }
}
```

In the above code `Die` is leaking two pieces of information:
* The die faces are modelled as integer numbers
* A die wins over another if the former `value` is greater than the latter

If any of the above two change, we will need to adjust `Game`  as its `play` method uses that information.
An example of such changes is a new rule by which the number 3 wins over any other number.   
However, we can prevent those unintended changes in `Game` by not leaking information out of `Die` as follows.

```kotlin
class Die {
  private var value = 1

  fun roll() {
    value = (1..6).random()
  }

  fun winner(player2Die: Die): String {
    if(value > player2Die.value) {
      return "Player 1 wins"
    }
    if(value < player2Die.value) {
      return "Player 2 wins"
    }
    return "Draw"
  }
}

class Game{

  fun play() {
    val player1Die = Die()
    val player2Die = Die()

    print(player1Die.winner(player2Die))
  }
}
```

We are now free to change the internal implementation of `Die` without interfering with `Game` or any other part of the codebase.

As a final note, leaking information can be subtle and can occur simply by picking the wrong name.

<br/>

#### Recommended reads
* [Encapsulation, chapter 10 of Clean Code - Robert C. Martin](https://www.goodreads.com/book/show/3735293-clean-code)
* [Encapsulate Behavior,Not Just State - chapter 10 of of 97 Things Every Programmer Should Know - Kevlin Henney](https://www.goodreads.com/book/show/7003902-97-things-every-programmer-should-know)
* [Information Hiding, chapter 5 of A Philosophy of Software Design - John Ousterhout](https://www.goodreads.com/en/book/show/39996759-a-philosophy-of-software-design)
* [Decoupling and the Law of Demeter, chapter 5 of The Pragmatic Programmer - David Thomas, Andrew Hunt](https://www.goodreads.com/book/show/4099.The_Pragmatic_Programmer)
* [Object Calisthenics, William Durand](https://williamdurand.fr/2013/06/03/object-calisthenics/)
* [Tell Above, and Ask Below - Michael Feathers](https://michaelfeathers.typepad.com/michael_feathers_blog/2012/03/tell-above-and-ask-below-hybridizing-oo-and-functional-design.html)

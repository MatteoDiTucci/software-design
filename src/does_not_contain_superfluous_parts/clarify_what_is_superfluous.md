## Clarify what is superfluous
Sometimes the [you aren't going to need it](#You-aren't-going-to-need-it) principle is abused, leading to code that does
not pass all tests. This usually happens in the form of assuming specifications. Let's consider the following example where we are required
to build a piece of code that:
* stores fruits by their names
* does not store a fruit name more than once
* given a fruit name, returns true if it has been previously stored

```kotlin
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
```

The above code works just fine given the specifications. However, we might argue that `fruits` implemented as a list
is not very performant and that we could make both `contains` and `storeFruit` faster by simply using a set.
It is a mistake to dismiss such thought appealing to the [you aren't going to need it](#You-aren't-going-to-need-it) principle.
In fact, we would be assuming a specification about performance that is currently unknown. We should instead clarify the
expected performance of `FruitInventory`: if it is irrelevant then the code is as good as it is, otherwise we need
to change it as follows.

```kotlin
class FruitInventory {
  private val fruits = HashSet<String>()

  fun storeFruit(name: String) {
    fruits.add(name)
  }
  
  fun contains(name: String): Boolean {
    return fruits.contains(name)
  }
}
```

Performance specifications aside, the above code using `HashSet` is superior as it better expresses intent. We could even
argue that `HashSet` approach satisfies the specifications just fine and we aren't going to need all the for-loop complexity
introduced by the `ArrayList` approach.

[1] [The full quote from Donald Knuth was: "We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil. Yet we should not pass up our opportunities in that critical 3%."](https://dl.acm.org/toc/csur/1974/6/4)
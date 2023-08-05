## Expand and contract
When we change the signature of a public method, we need to adjust all its client code accordingly. 
In big codebases this implies a single, time-consuming refactoring which can lead to version control conflicts (e.g. git merge conflicts)
with other developers. However, we can break down this big refactoring into smaller steps by leveraging a technique called expand and contract.
Let's take the following code as an example:

```kotlin
import java.math.BigDecimal

class Price(private val amount: BigDecimal) {

    fun discountedOf(percentage: BigDecimal): Price { 
        val discount = amount.multiply(percentage.divide(BigDecimal(100)))
        return Price(amount.subtract(discount))
    }
}
```

The `percentage` input parameter of the method `discountedOf` can be problematic as there is no guarantee for it to be between 0 and 100.
For this reason, we want to introduce a class `Percentage` so that the signature of the method `discountedOf` can become `discountedOf(percentage: Percentage)`.
Let's see how we can do this in small steps with the expand and contract technique.

### 1. Expand

Instead of modifying the signature of the method `discountedOf` right away, we add a new method `newDiscountedOf` which has the desired new signature:

```kotlin
import java.math.BigDecimal

class Price(private val amount: BigDecimal) {

    fun newDiscountedOf(percentage: Percentage): Price {
        val discount = percentage.of(amount)
        return Price(amount.subtract(discount))
    }

    fun discountedOf(percentage: BigDecimal): Price {
        val discount = amount.multiply(percentage.divide(BigDecimal(100)))
        return Price(amount.subtract(discount))
    }
}

class Percentage(private val percentage: BigDecimal) {
    init {
        require(percentage in BigDecimal(0)..BigDecimal(100) ) {
            "Percentage must be between 0 and 100"
        }
    }

    fun of(amount: BigDecimal): BigDecimal {
        return amount.multiply(percentage.divide(BigDecimal(100)))
    }
}
```

At this point for the client code nothing has changed: everybody still use the method `discountedOf` while `newDiscountedOf`
is dead code.

### 2. Migrate clients one by one
Now we can migrate the client code. However, instead of doing it all at once, we can refactor one client at a time.
In this way we can gradually increase the usage of `newDiscountedOf` while reducing the one of `discountedOf`.
Each client migration can be a standalone code revision (git commit), so the changes are small and incremental while 
the entire codebase keeps working as expected.

### 3. Contract
When all clients have been migrated, the method `discountedOf` is now dead code, so we can delete it and rename the method
`newDiscountedOf` into `discountedOf`. The final result looks like the following:

```kotlin
import java.math.BigDecimal

class Price(private val amount: BigDecimal) {

    fun discountedOf(percentage: Percentage): Price {
        val discount = percentage.of(amount)
        return Price(amount.subtract(discount))
    }
}

class Percentage(private val percentage: BigDecimal) {
    init {
        require(percentage in BigDecimal(0)..BigDecimal(100) ) {
            "Percentage must be between 0 and 100"
        }
    }

    fun of(amount: BigDecimal): BigDecimal {
        return amount.multiply(percentage.divide(BigDecimal(100)))
    }
}
```

The end result is the same as we would have done it in one shot. However, we made the refactoring more maneageable by
breaking it down into smaller steps.

### Further applications
Expand and contract is very useful to avoid merge conflicts, no matter if you are using [trunk base development or feature branches](https://martinfowler.com/articles/branching-patterns.html).
Especially with trunk based development, it plays very well with the use of [feature toggles](https://martinfowler.com/articles/feature-toggles.html).

Furthermore, despite the above example is about a class method, the same approach is applicable to api contracts.
Let's assume for instance a POST REST API with the following body:

```json
{
  "price" : 72.9,
  "discount" : 15
}
```

Then, let's assume we want to change `"discount"` from an integer to a percentage expressed as a decimal between 0 and 1.
Using the expand and contract approach, we would first expand:
```json
{
  "price" : 72.9,
  "discount" : 15,
  "discount_percentage" : 0.15
}
```

Then migrate all clients to use `"discount_percentage"` instead of `"discount"` and finally contract:
```json
{
  "price" : 72.9,
  "discount_percentage" : 0.15
}
```

Last but not least, we can use expand and contract also for database schema migrations.

<br/>  

#### Recommended reads
* [Surviving continuous deployment in distributed systems - Valentina Servile](https://oooops.dev/2021/07/30/surviving-continuous-deployment-in-distributed-systems/)
* [Refactoring Databases: Evolutionary Database Design - Scott Ambler, Pramod Sadalage](https://www.goodreads.com/book/show/161302.Refactoring_Databases)
* [Parallel change - Scott Ambler, Joshua Kerievsky](https://www.infoq.com/presentations/The-Limited-Red-Society/)
* [Parallel change - Danilo Sato](https://martinfowler.com/bliki/ParallelChange.html)

#### Teach me back
I really appreciate any [feedback](../introduction/introduction.html#teach-me-back) about the book and my current understanding of software design.
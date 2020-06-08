---
title: TIL about Operator Overloading in Kotlin and the Invoke Operator
# featured_image: /images/article-directory/featured-image.png
excerpt: Kotlin lets us define custom behaviour for operators (e.g. +, ==or *). We can add mathematical or logical semantics for how operators behave with various types. We can either implement these behaviours in a class as a member function (handy for classes that we own), or externally, as an extension function (for types outside of our control).

---

Kotlin lets us define custom behaviour for [operators](https://en.wikipedia.org/wiki/Operator_%28computer_programming%29) (e.g. +, ==or *). We can add mathematical or logical semantics for how operators behave with various types.

We can either implement these behaviours in a class as a member function (handy for classes that we own), or externally, as an extension function (for types outside of our control).

First, let’s see what overloading _is_.

##  What is overloading?

Overloading functions is a practice which allow us to provide multiple functions with the same name (in the same scope), but with different signatures. This is desirable when the behaviour is the same or similar but the implementation has to be different for different types.

Consider the following class, which has a single function that’s used to sum two `Int` values together:

```kotlin
class Summer {

    fun sum(a: Int, b: Int): Int {
        return a + b
    }
}
```

We can add a second function with the same name, in the same scope (in this class), but only if we change the parameters to alter the signature:

```kotlin
class Summer {

    fun sum(a: Int, b: Int): Int {
        return a + b
    } 
  
    fun sum(a: Money, b: Money): Money {
        return Money(a.value + b.value)
    } 
}
```

Now we can say that we’ve overloaded the “sum” function; whenever we call sum the function that it calls will depend on the types of the parameters we pass.

## What is operator overloading in Kotlin?

Operator overloading is similar. Operators like minus, plus or equals have been defined to work with a subset of predefined types.

Let’s consider the minus function which works with some types, like `Int`:

```
minus(a: Int, b: Int)
```

or

```
a - b
```

where `a` and `b` are of type `Int`.

We learned at the start that we can overload operators by including them inside classes as member functions or outside classes as extension functions.

The minus operator has [multiple overloads defined as **member functions**](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-int/minus.html) in the `Int` class:

```kotlin
operator fun minus(other: Byte): Int
operator fun minus(other: Short): Int
operator fun minus(other: Int): Int
operator fun minus(other: Long): Long
operator fun minus(other: Float): Float
operator fun minus(other: Double): Double
```

Let’s add another for a type that we define.

## Overloading an operator for a new type

We have a data class that represents a tub (box?) of ice cream. It has one value, which is the number of scoops remaining in the tub (usually a low number for me):

```kotlin
data class IceCreamTub(val remainingScoops: Int)
```

We want:

```kotlin
minus(a: IceCreamTub, b: Int)
```

We saw the overloading example with member functions above for `Int`, let’s demonstrate the same but using an extension function:

```kotlin
operator fun IceCreamTub.minus(scoops: Int): IceCreamTub {
    if (remainingScoops == 0) {
        throw NotEnoughIceCreamException()
    }
    return IceCreamTub(remainingScoops - scoops)
}
```

This will allow us to do something like this:

```kotlin
val iceCream = IceCreamTub(4) - 3
println(iceCream) // prints "IceCreamTub(remainingScoops=1)"
```

Could we do the opposite?

```kotlin
val something = 10 - IceCreamTub(4) // doesn't compile
```

It doesn’t compile because there’s no function minus defined on `Int` that has `IceCreamTub` as a parameter.

Let’s add it with an extension function (we can’t add a member function to the `Int` class directly):

```kotlin
operator fun Int.minus(iceCreamTub: IceCreamTub)...
```

Oh. I actually don’t know what the return type should be for this function. It’s nonsense!

## When should we (not) overload operators?

>Problems, and critics, to the use of operator overloading arise because it allows programmers to give operators completely free functionality, without an imposition of coherency that permits to consistently satisfy user/reader expectations

from [C++ Programming/Operators/Operator Overloading](https://en.wikibooks.org/wiki/C%2B%2B_Programming/Operators/Operator_Overloading). No “imposition of coherency” mean that the writer is unrestricted to do what they like.

Did it make sense for us to overload the minus operator for IceCream ? The two units aren’t even comparable to each other so maybe it didn’t make sense in the first place.

```kotlin
fun IceCreamTub.subtractScoops(numberOfScoops: Int): IceCreamTub
```

A regular function with a nicer name would serve us better.

We should be careful about overloading operators with types that might cause bewilderment (like with the Int and IceCreamTub combination) or worse, misunderstanding.

<center>
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">A common criticism of operator overloading is that it allows developers to create code that does unexpected things. For example: n/=2 means n=n/2, but (in C++), path /= &quot;/myfile.txt&quot; means &quot;path = path + &quot;/myfile.txt&quot;</p>&mdash; Princess Nicole (@LadyNikoleta) <a href="https://twitter.com/LadyNikoleta/status/1017463029640790016?ref_src=twsrc%5Etfw">July 12, 2018</a></blockquote>

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">It is my opinion that this does not mean operator overloading is bad, but just that this was probably a poor choice to use it.</p>&mdash; Princess Nicole (@LadyNikoleta) <a href="https://twitter.com/LadyNikoleta/status/1017463346197434369?ref_src=twsrc%5Etfw">July 12, 2018</a></blockquote>
</center>

## Useful instances of overloading

If you’ve used the Kotlin for even a few weeks, you’ll likely have come across instances of operator overloading already.

The `Collection` type overloads the `plus` operator. It lets us add elements from an `Iterable<T>` to a `Collection<T>`.

```kotlin
// Collection<T>.plus(elements: Iterable<T>): List<T>

val a = listOf(1, 2, 3)
val b = listOf(4, 5, 6)
println(a + b) // prints [1, 2, 3, 4, 5, 6]
```

The `plus` operator is further overloaded to support appending single elements:

```kotlin
// Collection<T>.plus(element: T): List<T>

val appendedElements = a + b + 7 + 8 + 9
println(appendedElements) // prints [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

And it even works with collections of different types — in this case, the type of mix will be `List<Any>`:

```kotlin
val a = listOf(1, 2, 3)
val b = listOf("four", "five", "six")
val mix = a + b

println(mix) // prints [1, 2, 3, four, five, six]
```

Have a look at the [full list of operators that can be overloaded on the Kotlin Language documentation site](https://kotlinlang.org/docs/reference/operator-overloading.html).

## The invoke() operator

The `invoke()` operator allows instances of your classes to be called as functions.

Let’s create a class whose sole responsibility is taking a scoop of ice cream from the ice cream tub, then return the scoop and the tub (with the updated contents).

```kotlin
data class IceCream(val scoops: Int)
data class ScoopResult(val iceCream: IceCream?, val iceCreamTub: IceCreamTub)

class ScoopAction {

    fun scoop(iceCreamTub: IceCreamTub): ScoopResult {
        val scoops = iceCreamTub.remainingScoops
        if (scoops == 0) {
            return ScoopResult(null, iceCreamTub)
        }
        return ScoopResult(IceCream(1), IceCreamTub(scoops - 1))
    }
}
```

How would we use this?

```kotlin
val scoopAction: ScoopAction

fun eatIceCream() {
    val scoopResult = scoopAction.scoop(iceCreamTub)
    // TODO: eat it
}
```

Our `ScoopAction` is only designed to do one thing — it’s a little clunky to have to call a function on a class that _solely represents_ an action.

Instead, we could overload the `invoke()` operator:

```kotlin
class ScoopAction {

    operator fun invoke(iceCreamTub: IceCreamTub): ScoopResult {
        ...
    }
}
```

The only differences are the addition of the `operator` keyword and usage of `invoke` as the function name.

```kotlin
val scoop: ScoopAction

fun eatIceCream() {
    val scoopResult = scoop(iceCreamTub)
    // TODO: eat it
}
```

We can further overload the same operator with more parameters:

```kotlin
class ScoopAction {

    operator fun invoke(iceCreamTub: IceCreamTub): ScoopResult {
        return invoke(iceCreamTub, 1)
    }
    
    operator fun invoke(iceCreamTub: IceCreamTub, scoops: Int): ScoopResult {
        ...
    }
}
```

Parentheses are translated to calls to `invoke` with appropriate arguments.

## Have I been using the invoke() operator already?

The most common place we encounter the `invoke()` operator is with lambdas!

Lambdas are compiled to instances of the [Function interfaces](https://github.com/JetBrains/kotlin/blob/d3156c33be09fae459df1247476a401605ae92ae/libraries/stdlib/jvm/runtime/kotlin/jvm/functions/Functions.kt) which all **overload the `invoke()` operator**:

```kotlin
public interface Function0<out R> : Function<R> {
    public operator fun invoke(): R
}

public interface Function1<in P1, out R> : Function<R> {
    public operator fun invoke(p1: P1): R
}

public interface Function2<in P1, in P2, out R> : Function<R> {
    public operator fun invoke(p1: P1, p2: P2): R
}

...
```

This lets us do the following:

```kotlin
val square = { value: Int -> value * value }

println(square(5)) // prints 25
```

Instead of:

```kotlin
println(square.invoke(5)) // also prints 25
```

## So… should I use this in my project?

We looked at what operator overloading is and how to do it in Kotlin. We see that the operators serve as a shorthand for function calls, and consequently anything written with an operator can be written explicitly with the corresponding function.

We write `1 + 2` instead of `1.plus(2)` because it’s short, commonly understood and therefore easier to read, so we should apply the same logic when we’re deciding whether to overload operators with our own types:

- is it clear what the behaviour _should_ be?
- would it be used often enough in your project to warrant the shorthand?

The `invoke()` operator is a little different. It still provides you with a shorthand, but has no inherent behaviour/meaning of its own — it lends your classes the property of being actionable. If your class:

- represents a single action
- is named for a specific action

then go for it!

It doesn’t require any extra lines of code, it supports and highlights the responsibility of a single action for your class and it reduces verbosity at the call site.

As always, questions and comments welcome either here or [on Twitter](https://twitter.com/ataulm)!

Thanks [Daniele](https://twitter.com/fourlastor), [Nicole](https://medium.com/@borrelli) and [Maria](https://medium.com/@marianeum) for reviewing, as well as [Rebecca](https://medium.com/@riggaroo) and [Florina](https://medium.com/@florina.muntenescu) for discussing.

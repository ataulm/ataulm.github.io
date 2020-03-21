---
title: TIL that you can use the Invoke Operator on Companion Objects too
# featured_image: /images/article-directory/featured-image.png
excerpt: Kotlin lets us define custom behaviour for operators (e.g. +, ==or *). We can add mathematical or logical semantics for how operators behave with various types. We can either implement these behaviours in a class as a member function (handy for classes that we own), or externally, as an extension function (for types outside of our control).

---

In [a previous article]({% post_url 2018-08-10-operator-overloading %}), we looked at overloading the `invoke()` operator, how to do it, and why we might want to.

I was asked to highlight that you can use the `invoke()` operator on a class’s companion object as a way to have a “validation constructor”.

<center>
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">For Part 2 I&#39;d love to see my favorite pattern for invoke: putting it on companion objects and hide the real constructor, so you can get &quot;validation constructors&quot; that return Option or Either if the parameters aren&#39;t correct :D <a href="https://t.co/LJ1R8rdaQ9">pic.twitter.com/LJ1R8rdaQ9</a></p>&mdash; Paco (@pacoworks) <a href="https://twitter.com/pacoworks/status/1027945767938469890?ref_src=twsrc%5Etfw">August 10, 2018</a></blockquote>
</center>

First of all, cool! I didn’t realise you could do this, but today I learned and it makes sense:

>a companion object is initialized when the corresponding class is loaded (resolved), matching the semantics of a Java static initializer.

from the [Kotlin Language documentation page on Object Expressions and Declarations](https://kotlinlang.org/docs/reference/object-declarations.html).

An initialized companion object is still an instance of a class (of type `MyClass.Companion`) with which you can overload operators.

### Overloading invoke() in a companion object

Let’s see this in action. I haven’t included any third-party libraries, so while the tweet above refers to an `Option<User>` I’ll just be using Kotlin’s optional type:

```kotlin
class IceCream private constructor(private val scoops: Int) {

    companion object {

        operator fun invoke(): IceCream? {
            return invoke(0)
        }

        operator fun invoke(scoops: Int): IceCream? {
            return if (scoops < 0) {
                null
            } else {
                IceCream(scoops)
            }
        }
    }
}
```

And what does it look like to use?

```kotlin
val noIceCream = IceCream()
val someIceCream = IceCream(1)
val moreIceCream = IceCream(2)

// or

val invalidIceCream = IceCream(-1)
```

### Should I overload the invoke() operator in a companion object?

I’m not a fan—[I would be slightly astonished](https://en.wikipedia.org/wiki/Principle_of_least_astonishment) if I encountered this in a project.

For me, because it looks so much like a constructor invocation, I would expect that these were initializing objects of type IceCream or crashing at runtime because of some validation error inside the constructor.

Are there any hints that the type of these values is **not** `IceCream`? Yes!

- if you explicitly specify the type, or hover over the value, you’ll see that they’re of type `IceCream?` and not `IceCream`
- when you try to use it and get a type error, you’ll realise it’s not `IceCream`
- If you **hold cmd/ctrl and hover over the `()`** with your mouse, you’ll see it’s clickable and will take you to the declaration of the `invoke()` operator

### So what alternatives are there?

You can use factory classes or even named functions in your companion objects to be more explicit.

<center>
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">This enforces that all objects are well formed, something you&#39;d have to check regardless before or after calling a *real* constructor. If you prefer to give this validation constructor a name and still hide the real constructor, that&#39;s good too. I&#39;ve not seen that much pushback.</p>&mdash; Paco (@pacoworks) <a href="https://twitter.com/pacoworks/status/1028248385067864066?ref_src=twsrc%5Etfw">August 11, 2018</a></blockquote>
</center>

I spoke with Paco about these factories returning Option<MyClass> instead of MyClass? or even throwing an exception in case of invalid parameters and he shared:

>“This conversation on Twitter highlighted one improvement we could have in our codebases. By moving to validated constructors, we can avoid a whole set of runtime errors.
>
>“These errors would be highly conditional on the parameters on the constructor, which means difficult to trigger while debugging, and would need to be documented and taken into account when refactoring.
>
>“By simply creating a new validated constructor that returns null or an `Option` (i.e. [Arrow](https://arrow-kt.io/docs/datatypes/option/)/[Koptional](https://github.com/artem-zinnatullin/koptional) on a failure), we’re conveying the needs for our object to be constructed, and those will be explicitly mandated across future calls and refactors without requiring users to check the documentation.”

Thanks [Paco](http://pacoworks.com/)!

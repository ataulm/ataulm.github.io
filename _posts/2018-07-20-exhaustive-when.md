---
title: "TIL: when is \"when\" exhaustive?"
# featured_image: /images/article-directory/featured-image.png
excerpt: Kotlin’s when works similarly to Java’s switch. Once upon a time, I heard that when forces you to specify all the branches (when all the branches can be known by the compiler, with enums or sealed-classes), but I didn’t find this to be the case.

---

Kotlin’s when works similarly to Java’s switch. Once upon a time, I heard that when forces you to specify all the branches (when all the branches can be known by the compiler, with enums or sealed-classes), but I didn’t find this to be the case.

IDEs like Android Studio might be able to give you a hint (as it did with switch cases), reminding you to be explicit:

![“when” block with hints to add “else” branch or add remaining branches](/images/exhaustive-when/hint.png)

But the [documentation for when](https://kotlinlang.org/docs/reference/control-flow.html) makes it clearer:

>If [`when`] is used as an expression, the value of the satisfied branch becomes the value of the overall expression [... and] the `else` branch is mandatory, unless the compiler can prove that all possible cases are covered with branch conditions

So, if we use it as an expression, we get a compiler error!

![“when” block being assigned to a value as an expression, with error telling us to add the remaining branches or an “else” branch](/images/exhaustive-when/error.png)

Sometimes you don’t want to use it as an expression but you still want to explicitly handle all the cases. With `switch`, we’d often use the `default` case to express our intent:

```java
void onNext(Result result) {
    switch (result.type) {
        case COMPLETE:
            show(result.data)
            break;
        case LOADING:
            showLoading()
        case ERROR:
            showError(result.error)
            break;
        default:
            throw new IllegalArgumentException("unknown result type: " + result);
    }
}
```

If we later update the `Result.Type` enum without updating this logic, we’ll crash. This is preferable to unexpected behaviour.

In Kotlin, we have the else branch instead of a `default` case, so we could do the same thing with an Exception there.

However, we can make it a **compile-time check** if we treat the when as an expression with an empty let block:

![“when” block, with error telling us to add the remaining branches or an “else” branch, because we’re trying to use the block as an expression with our empty “let”](/images/exhaustive-when/compile-check.png)

This doesn’t look great though, and it’s not clear what we’re doing — if I were to read this in the future, I’d probably delete the `.let {}` thinking it’s doing nothing!

Instead, you can add an extension property with a name that helps explain the purpose:

```kotlin
val <T> T.exhaustive: T
    get() = this
```

This property has a custom getter which returns the object itself, so if we use it on a when block, it’s treated as an expression and the compiler will force us to specify all cases.

![“when” block, with error telling us to add the remaining branches or an “else” branch, because we added `.exhaustive` making the compiler think we’re using it as an expression](/images/exhaustive-when/compile-check-exhaustive.png)

---

I [learned this from the Plaid 2.0 project](https://github.com/nickbutcher/plaid/commit/a33ba90e51a4c48fe4acc7d883ed2b160e5b03b8#diff-c311214d70ffbaa6d505fac1f51fd2b9R35), which is currently under a re-write effort.

Thanks to the team for being patient with my comments and questions!

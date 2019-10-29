---
title: Resolving View Attributes on Android
featured_image: '/images/resolving-view-attrs/feature.png'
excerpt: When a view is inflated from XML, where does Android look to determine the value of the view's attributes? We’ll go through an example showing how to define and use custom attributes, and how Android will resolve them when the view is inflated.
---

![](/images/resolving-view-attrs/feature.png)

When we write custom views, it’s not uncommon for us to define new attributes so that we can allow instances to be configured. Let’s look at how we define and use them, and then we’ll look into how attributes are resolved.

We’ll make a `SpottyFrameLayout` which draws spots on top of its children, and define a couple of custom attributes, `spotColor` and `spotSize`, and look at different ways we can use these attributes.

### Defining and using custom attributes

Let’s give a quick overview, then we’ll revisit parts. First up, we define our custom view:

```kotlin
class SpottyFrameLayout(context: Context, attrs: AttributeSet)
    : FrameLayout(context, attrs)
```

Our custom attributes are defined in `res/main/values/attrs.xml`:

```xml
<resources>
    <declare-styleable name="SpottyFrameLayout">
        <attr name="spotColor" format="color" />
        <attr name="spotSize" format="dimension" />
    </declare-styleable>
</resources>
```

In our layout file, we can now use our custom view, with custom attributes:

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.example.SpottyFrameLayout   
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:spotColor="@color/material_red_900"
    app:spotSize="16dp">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:text="Hello Android!" />

</com.example.SpottyFrameLayout>
```

resolve these values in code:

```kotlin
class SpottyFrameLayout(context: Context, attrs: AttributeSet) : FrameLayout(context, attrs) {

    init {
        val typedArray = context.obtainStyledAttributes(attrs, R.styleable.SpottyFrameLayout)
        val spotSize = typedArray.getDimension(R.styleable.SpottyFrameLayout_spotSize, 0f)
        val spotColor = typedArray.getColor(R.styleable.SpottyFrameLayout_spotColor, 0)
        typedArray.recycle()
        ...
```

and use these values to draw some lovely spots (code not shown):

![](/images/resolving-view-attrs/spotty_framelayout.png)

### Finding attribute values

The `View` constructor has a few overloads. When a view is created as a result of XML inflation, the two-argument constructor is used, where `AttributeSet` will contain all of the attributes that have been defined on the view in XML.

![](/images/resolving-view-attrs/attrs.png)

This isn’t limited to the attributes that we’ve defined explicitly either. Even though we declared our attributes in a styleable (`res/main/values/attrs.xml`), it’s not necessary. This would compile:

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.example.SpottyFrameLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    undeclaredAttr="value for undeclared attr">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:text="Hello Android!" />

</com.example.SpottyFrameLayout>
```

and we'd even be able to get it from the attribute set that's passed in the constructor. So why do we declare custom attributes, and why do we declare them in styleables?

There’s a few reasons! We’ll get contextual autocomplete in XML layouts when we start to write because Android Studio knows (from the styleable’s name) that `SpottyFrameLayout` has two custom attributes with specific types (color and dimension, respectively).

![](/images/resolving-view-attrs/attr_autocomplete.png)

Using a styleable means we can use `Resources.Theme.obtainStyledAttributes(...)` which gives us a `TypedArray`, and these both contribute to a more convenient experience than using the `AttributeSet` class directly.

```kotlin
val typedArray = context.obtainStyledAttributes(attrs, R.styleable.SpottyFrameLayout)
```

`Resources.Theme.obtainStyledAttributes` fills a `TypedArray` with the values we requested (in the styleable). For each attribute, it’ll first check whether it’s available in the attribute set, using that value if it’s there. 

If it can’t find it, it’ll check the style resource (if one was set on the view).

![](/images/resolving-view-attrs/style-res.png)

The attributes from the style resource aren't resolved and "imported" into the attribute set. If they were, then we would be unable to differentiate between the values we set on the view and the values from the style resource. Since they're not, we can override attributes specified in the style by setting it on the view itself in XML. 

If the requested attribute isn't found in either the `AttributeSet` or the style resource, then it’ll check for the _default style resource_, specified by either the `defStyleAttr` or `defStyleRes`. These are the three- and four-argument `View` constructors and they're only used by the view itself (or a subclass).

- `defStyleAttr` is defined in the code for the view, and it allows users to specify a (default) style resource for that type of view in their theme. If the view provides `-1` (or often `0`) as the value for `defStyleAttr`, or we don't provide a value for `defStyleAttr` in the theme, then `defStyleRes` is checked.
- `defStyleRes` which is a style resource that contains default attribute values from the author of the view. When we provide a value for `defaultStyleAttr` we often want our style to extend from the one specified by `defStyleRes` so that we only override particular attributes.

If the requested attribute isn’t found in the default style resource, then it’ll check the theme(s), starting with the overlays on the outside, working its way inside until it either finds the requested attribute or returns null.

### Using a TypedArray

So now we’ve got a `TypedArray`, filled with values (or nulls) for the attributes we requested from the styleable we passed, and we can query the array for values. It's a wrapper around an array, so we could access the values like so:

```kotlin
init {
    val typedArray = context.obtainStyledAttributes(attrs, R.styleable.SpottyFrameLayout)
    val spotColor = typedArray.getColor(2, 0)
    val spotSize = typedArray.getDimensionPixelSize(3, 0)
    typedArray.recycle()
}
```

where the `0` is a default value if the value at the specified index is null.

Using an integer index directly isn't very meaningful though (and it's even worse, because the indices correspond to the _alphabetized_ position of our attributes, not the order that we defined them). Instead, we should use the styleable attributes that we defined directly:

```kotlin
init {
    val typedArray = context.obtainStyledAttributes(attrs, R.styleable.SpottyFrameLayout)
    val spotColor = typedArray.getColor(R.styleable.SpottyFrameLayout_spotColor, 0)
    val spotSize = typedArray.getDimensionPixelSize(R.styleable.SpottyFrameLayout_spotSize, 0)
    typedArray.recycle()
}
```

The handy thing about using the functions from `TypedArray` is that it’ll take care of resource resolution—the value for a color attribute could be a color resource `@color/material_red_900` or it could be a hexcode, but the return value will be a color integer.

The `TypedArray` instance is obtained from a pool, so it’s important that we recycle it (mark it as available for reuse) after we've pulled all the values we need.

### What’s next?

The important takeaway from this post is that an attribute’s value is determined by checking 4 places in a specific order:

1. Any attribute values in the given `AttributeSet` (setting a value on the view in XML)
2. The style resource specified in the AttributeSet (setting a `style` on the view in XML)
3. The default style resource: specified in a theme by `defStyleAttr` or specified by `defStyleRes` (only checked if `defStyleAttr` is `-1` or `0` or isn’t set in the theme)
4. The base values in this theme

"Base" values in the theme is a bit of a misnomer too because we could have multiple themes (as overlays via `ContextThemeWrapper`). In this case, the overlays applied at the top/outer layer are checked first, and can override attributes from theme overlays/themes from the inner layers.

With this in mind, what’s next? We learned about `materialThemeOverlay` in the previous post and how we can add support to our custom views for this attribute. In our next post, we'll learn why it's _necessary_ to add support explicitly, and why it's not possible for the layout inflater to handle it on our behalf.

Please [let me know if you found this post helpful](https://twitter.com/ataulm/status/1188846659784118274), or if you have any comments or questions (or corrections!).

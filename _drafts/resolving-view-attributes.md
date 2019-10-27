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

### Resolving attributes in code

The `View` constructor has a few overloads. When a view is created as a result of XML inflation, the two-argument constructor is used, where `AttributeSet` will contain all of the attributes that have been defined on the view in XML.

This isn’t limited to the attributes that we’ve defined explicitly either. Even though we declared our attributes in a styleable (`res/main/values/attrs.xml`), it’s not necessary—this would compile:

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

and it would even be available for us to query via the `AttributeSet` that’s passed through to the constructor too. So why do we declare custom attributes, and why do we declare them in styleables?

There’s a few reasons! We’ll get contextual autocomplete in XML layouts when we start to write because Android Studio knows (from the styleable’s name) that `SpottyFrameLayout` has two custom attributes with specific types (color and dimension, respectively).

![](/images/resolving-view-attrs/attr_autocomplete.png)

Using a styleable means we can use `Resources.Theme.obtainStyledAttributes(...)` which gives us a `TypedArray`, and these both contribute to a more convenient experience than than using `AttributeSet` directly.

```kotlin
val typedArray = context.obtainStyledAttributes(attrs, R.styleable.SpottyFrameLayout)
```

`Resources.Theme.obtainStyledAttributes` fills a `TypedArray` with the values we requested (in the styleable). For each attribute, it’ll first check whether it’s available in the `AttributeSet`, using that value if it’s there. If it can’t find it, it’ll check the style resource that’s specified in the `AttributeSet` (if any). This is the reason we can override an attribute specified in the style by setting it on the view itself in XML.

If it can’t find the requested attribute in either the `AttributeSet` or the style resource, then it’ll check the next level for the requested attribute, which is either the `defStyleAttr` or `defStyleRes`.

- `defStyleAttr` is defined in the view, allowing users to specify a style resource in their theme. If the view provides 0 as the value for `defStyleAttr`, or the `defStyleAttr` hasn’t been used in the theme, then `defStyleRes` is checked.
- `defStyleRes` which is a default style resource that should be applied to the view

If the requested attribute isn’t found in that level, then it’ll check the theme(s), starting with the overlays on the outside, working its way inside until it either finds the requested attribute or returns null.

So now we’ve got a `TypedArray`, we can query it for values. The nice thing is that it’ll take care of resource resolution—the value for a color attribute could be a color resource `@color/material_red_900` or a hexcode, but the return value will be a color integer. Since it’s just a wrapper around an array, we could access the values with integers (`0` or `1` in our case because we’ve only got two attributes). But it’s not clear to read and it can be confusing since the indices correspond to the _alphabetized_ position of our custom attributes.

Instead, we use the styleable resources that we defined directly, which conveys a lot more meaning:

```kotlin
init {
    val typedArray = context.obtainStyledAttributes(attrs, R.styleable.SpottyFrameLayout)
    val spotColor = typedArray.getColor(R.styleable.SpottyFrameLayout_spotColor, 0)
    val spotSize = typedArray.getDimensionPixelSize(R.styleable.SpottyFrameLayout_spotSize, 0)
    typedArray.recycle()
}
```

### What’s next?

The important takeaway from this post is that an attribute’s value is determined by checking 4 places in a specific order:

1. Any attribute values in the given `AttributeSet` (setting a value on the view in XML)
2. The style resource specified in the AttributeSet (setting a `style` on the view in XML)
3. The default style resource: specified in a theme by `defStyleAttr` or specified by `defStyleRes` (only checked if `defStyleAttr` is 0 or isn’t set in the theme)
4. The base values in this theme

"Base" values in the theme is a bit of a misnomer too. We can use `ContextWrapper` to _overlay_ themes on top of each other. In this case, the overlays applied at the top/outer layer are checked first, and can override attributes from theme overlays/themes in the base layers.

With this in mind, what’s next? We learned about `materialThemeOverlay` in another post and how we can add support to our custom views for this attribute. It works using a `ContextThemeWrapper` passing the value of `materialThemeOverlay` (if specified).

In our next post, we’ll use what we’ve learned here to demonstrate why it’s necessary to do add support for this attribute in our custom views explicitly, and show how we can add limited support for mix and match styles.

---
title: What's the difference between a theme attribute and a view attribute?
featured_image: /images/backporting-view-attrs/multiple-inheritance.png
excerpt: Themes are maps of "theme attributes" to resources, while styles are maps of "view attributes" to resources. In this post, we'll look at the difference between theme attributes and view attributes.
---

We read about the difference between [themes and styles in this post](https://medium.com/androiddevelopers/android-styling-themes-vs-styles-ebe05f917578) by Nick Butcher and Chris Banes. Themes are maps of "theme attributes" to resources, while styles are maps of "view attributes" to resources. In this post, we'll look at the difference between theme attributes and view attributes.

## What's an attribute?

An attribute is a _named descriptor_ for a value we want to set in XML.

It's a resource itself and we can define them with the `<attr>` tag in `res/values` (like strings, dimensions and colors):

```xml
<resources>
    <attr name="myAppTextColor" format="color" />
</resources>
```

We can set this attribute in a layout or in a style/theme:

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:myAppTextColor="@color/red" />
```

In a layout, it'll be ignored. `TextView` doesn't know this attribute, nor does any other view.

However, if we set a value for this attribute in our theme:

```xml
<style name="Theme.MyApp" parent="Theme.MaterialComponents">
    <item name="myAppTextColor">@color/red</item>
</style>
```

we can _query_ it, programmatically:

```kotlin
val myAppTextColor = context.resolveColorAttr(R.attr.myAppTextColor)
textView.setTextColor(myAppTextColor) // red text!

// ...

@ColorInt fun Context.resolveColorAttr(@AttrRes attr: Int, @ColorInt defaultColor: Int = 0): Int {
    val typedArray = obtainStyledAttributes(intArrayOf(attr))
    // index = 0 because there's only one attribute in our typed array
    val color = typedArray.getColor(0, defaultColor)
    typedArray.recycle()
    return color
}
```

or use the `?attr` syntax in XML, to have it resolved for us:

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:textColor="?attr/myAppTextColor" />
```

`@attr/myAppTextColor` is a theme attribute.

## Declaring attributes for a view

We learned in [_Resolving view attributes on Android_]({% post_url 2019-10-28-resolving-view-attributes %}) that the correct way to define a custom attribute for a view includes declaring it as part of a set of attributes that the view will use (a "styleable"):

```xml
<resources>
    <!-- Styleables are named after their associated view -->
    <declare-styleable name="SpottyFrameLayout">
        <attr name="spotColor" format="color" />
        <attr name="spotSize" format="dimension" />
    </declare-styleable>
</resources>
```

```xml
<com.example.SpottyFrameLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:spotColor="@color/red"
    app:spotSize="16dp">

    <!-- ... -->
```

`@attr/spotColor` and `@attr/spotSize` are view attributes.

## So what's the difference?

A view attribute is an attribute that describes the desired value for a _particular view property_. We can recognise them because they're the attributes which are included in a styleable declaration.

A theme attribute is the other one. These usually define abstract concepts which don't map to a specific view property. For example, `TextView` uses `@android:attr/textColorPrimary` as its _default_ text color which we can override the value for in our theme, and it'll affect all textviews that don't have their text color set explicitly.
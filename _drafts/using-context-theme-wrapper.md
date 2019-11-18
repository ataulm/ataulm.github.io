---
title: Using Context Theme Wrapper on Android
featured_image: /images/using-context-theme-wrapper/feature.png
excerpt: TODO - context theme wrapper
---

`ContextThemeWrapper` allows us to modify or replace the theme of another context. This can be handy when we want to apply a theme overlay to a view that we're inflating at runtime.

Let's have a look at why and how we'd do this, using an alert dialog as an example.

### Alert dialogs in a Material world

[Material Components for Android](https://github.com/material-components/material-components-android) doesn't include an alert dialog implementation. Instead, `MaterialAlertDialogBuilder` is used to apply Material theming (color, typography and shape) to the AppCompat implementation.

```kotlin
MaterialAlertDialogBuilder(context)
        .setTitle("Verify your identity")
        .setMessage("Log in using your fingerprint")
        // ...
        .create()
```

The builder will use theme attributes to theme the dialog (in our theme, we've set `colorPrimary` to red):

![dialog where button text is blue](/images/using-context-theme-wrapper/01.png)

### Custom views in alert dialog

The builder lets us set a custom view as a component in the dialog.

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:id="@+id/fingerprintAuthIcon"
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:src="@drawable/ic_baseline_fingerprint_24"
        android:tint="?colorPrimary" />

    <TextView
        android:id="@+id/fingerprintAuthStateText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Touch the fingerprint sensor"
        android:textColor="?colorOnSurface" />

</LinearLayout>
```

```kotlin
val customView = LayoutInflater.from(context).inflate(R.layout.dialog_custom_view, ...)

MaterialAlertDialogBuilder(context)
        .setTitle("Verify your identity")
        .setMessage("Log in using your fingerprint")
        // ...
        .setView(customView)
        .create()
```

and since we referenced theme attributes (`?colorPrimary` and `?colorOnSurface`) for the icon tint and text color, these appear red too:

![dialog where icon and button text are red](/images/using-context-theme-wrapper/02.png)

### Changing the primary color

Let's change the primary color for alert dialogs so that we use blue instead of red, without affecting the rest of our app though. There are two ways we can achieve this with theme overlays.

In our theme, we can set the `materialAlertDialogTheme` attribute, pointing it to our theme overlay:

```xml
<style name="Theme.Demo" parent="Base.Theme.Demo">
    ...
    <item name="materialAlertDialogTheme">@style/Theme.Demo.Dialog</item>
</style>

<style name="Theme.Demo.Dialog" parent="ThemeOverlay.MaterialComponents.MaterialAlertDialog">
    <item name="colorPrimary">@color/material_blue_900</item>
</style>
```

Alternatively, we can pass the theme to the builder if we want to do it on a case-by-case basis.

```kotlin
MaterialAlertDialogBuilder(context, R.style.Theme_Demo_Dialog)
```

Either way, this is what we end up with:

![dialog where icon is red, button text is blue](/images/using-context-theme-wrapper/03.png)

Wait, why is our icon still red?

### Using the correct context

Our custom view is being inflated with a context that has `colorPrimary` set to red. The dialog builder, on the other hand, is either checking the theme for the `materialAlertDialogTheme` or is being passed it explicitly in the constructor.

We should use `ContextThemeWrapper`.

```kotlin
val dialogThemeContext = ContextThemeWrapper(requireContext(), R.style.Theme_Demo_Dialog)
val customView = LayoutInflater.from(dialogThemeContext).inflate(R.layout.dialog_custom_view, ...)
```

Now, our custom view will be inflated with a context where `colorPrimary` resolves to blue:

![dialog where icon and button text are blue](/images/using-context-theme-wrapper/04.png)

#### Aside: using the theme attribute

It's unlikely that all of our alert dialogs will be using a custom view, so we probably want to take advantage of the theme attribute, `materialAlertDialogTheme` to coupling this `ContextThemeWrapper` usage to a particular theme.

```kotlin
    ...
    val dialogTheme = context.resolveThemeAttr(R.attr.materialAlertDialogTheme)
    val dialogThemeContext = ContextThemeWrapper(requireContext(), dialogTheme)
}

private fun Context.resolveThemeAttr(@AttrRes attr: Int) = TypedValue().let { typedValue ->
    theme.resolveAttribute(attr, typedValue, true)
    check(typedValue.type == TYPE_REFERENCE)
    typedValue.resourceId
}
```

This means that whenever the value set in your theme changes, the `ContextThemeWrapper` will stay up-to-date.

Currently the `materialAlertDialogTheme` hasn't been made explicitly public, meaning that Lint will flag usages in code. I've opened [an issue for this here](https://github.com/material-components/material-components-android/issues/769).

### Conclusion


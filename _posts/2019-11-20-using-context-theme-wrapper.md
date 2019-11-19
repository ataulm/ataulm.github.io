---
title: Using Context Theme Wrapper on Android
featured_image: /images/using-context-theme-wrapper/feature.png
excerpt: A ContextThemeWrapper allows us to modify or overlay the theme of another context. Let’s have a look at how we’d do this, using an alert dialog as an example.
---
 
A `ContextThemeWrapper` allows us to modify or overlay the theme of another context. Let’s have a look at how we’d do this, using an alert dialog as an example.
 
### Alert dialogs in a Material world
 
[Material Components for Android](https://github.com/material-components/material-components-android) doesn’t include an alert dialog implementation. Instead, `MaterialAlertDialogBuilder` is used to apply Material theming (color, typography and shape) to the AppCompat implementation.
 
```kotlin
MaterialAlertDialogBuilder(context)
       .setTitle("Verify your identity")
       .setMessage("Log in using your fingerprint")
       // ...
       .create()
```
 
The builder will use the theme of the context given to the builder to theme the dialog (in our theme, we’ve set `colorPrimary` to red):

![dialog where button text is red](/images/using-context-theme-wrapper/01.png)
 
### Custom views in an alert dialog
 
The builder lets us provide a custom view as a component in the dialog.
 
```xml
<LinearLayout
   android:layout_width="match_parent"
   android:layout_height="match_parent">
 
   <ImageView
       android:id="@+id/fingerprintAuthIcon"
       android:layout_width="40dp"
       android:layout_height="40dp"
       app:srcCompat="@drawable/ic_baseline_fingerprint_24"
       app:tint="?attr/colorPrimary" />
 
   <TextView
       android:id="@+id/fingerprintAuthStateText"
       android:layout_width="match_parent"
       android:layout_height="wrap_content"
       android:text="Touch the fingerprint sensor"
       android:textColor="?attr/colorOnSurface" />
 
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
 
Since we referenced a theme attribute for the icon tint (`colorPrimary`), this appears red too:
 
![dialog where icon and button text are red](/images/using-context-theme-wrapper/02.png)
 
### Changing the primary color
 
Let’s change the primary color for alert dialogs to use blue instead of red, being careful not to affect the rest of our app. We can achieve this using theme overlays in two ways.
 
In our app’s theme, we can set  `materialAlertDialogTheme` to a theme overlay:
 
```xml
<style name="Theme.Demo" parent="Base.Theme.Demo">
   ...
   <item name="materialAlertDialogTheme">@style/ThemeOverlay.Demo.Dialog</item>
</style>
 
<style name="ThemeOverlay.Demo.Dialog" parent="ThemeOverlay.MaterialComponents.MaterialAlertDialog">
   <item name="colorPrimary">@color/material_blue_900</item>
</style>
```
 
This is similar to setting a default style for a widget, this theme overlay will be used for every Material alert dialog (unless otherwise overridden).
 
The other way we can do this is by passing a theme resource ID directly to the builder. This is useful if we want to modify a dialog’s theme on a case-by-case basis.

 
```kotlin
MaterialAlertDialogBuilder(context, R.style.ThemeOverlay_Demo_Dialog)
```
 
Whichever approach we take, this is what we end up with:
 
![dialog where icon is red, button text is blue](/images/using-context-theme-wrapper/03.png)
 
Wait, why is our icon still red?
 
### Using the correct context
 
Our custom view is still being inflated with a context that has `colorPrimary` set to red. This is where we could use a `ContextThemeWrapper`.
 
```kotlin
val dialogThemeContext = ContextThemeWrapper(context, R.style.ThemeOverlay_Demo_Dialog)
val customView = LayoutInflater.from(dialogThemeContext).inflate(R.layout.dialog_custom_view, ...)
```
 
Now, our custom view will be inflated with a context where `colorPrimary` resolves to blue:

![dialog where icon and button text are blue](/images/using-context-theme-wrapper/04.png)
 
#### Aside: using the theme attribute
 
Using the theme resource ID directly works, but then we need to keep this in sync with the theme used by the dialog. We could instead query the value of `materialAlertDialogTheme` from our theme:
 
```kotlin
   ...
   val dialogTheme = context.resolveThemeAttr(R.attr.materialAlertDialogTheme)
   val dialogThemeContext = ContextThemeWrapper(context, dialogTheme)
}
 
private fun Context.resolveThemeAttr(@AttrRes attr: Int) = TypedValue().let { typedValue ->
   theme.resolveAttribute(attr, typedValue, true)
   typedValue.resourceId
}
```
 
This means that whenever the value for `materialAlertDialogTheme` changes, the layout inflater will always use a `ContextThemeWrapper` with a matching theme.
 
Currently the `materialAlertDialogTheme` hasn’t been made explicitly public, meaning that Lint will flag usages in code. I’ve opened [an issue for this here](https://github.com/material-components/material-components-android/issues/769).
 
### When would we use this?

Cool! So we’ve seen how we could use a `ContextThemeWrapper` to overlay a theme programmatically. In this example, we could have achieved the same effect in the layout for our custom view:

```xml
<LinearLayout
   android:layout_width="match_parent"
   android:layout_height="match_parent"
   android:theme="?attr/materialAlertDialogTheme">

   ...
```

This works on Android 5 and above (or earlier if we’re using an AppCompat or MaterialComponents theme). The layout inflater will read the value of `android:theme` and use a `ContextThemeWrapper` in the same way we’ve done above.

There’s a couple of cases where `ContextThemeWrapper` is exclusively helpful. For example, [with data binding](https://github.com/google/iosched/blob/89df01ebc19d9a46495baac4690c2ebfa74946dc/mobile/src/main/java/com/google/samples/apps/iosched/ui/sessiondetail/SessionDetailFragment.kt#L114-L119), since `android:theme` can’t be used or in the case of `materialThemeOverlay`, being able to add support for theme overlays in default style resources without breaking support for `android:theme`.

In our next post, we’ll look at how `android:theme` was backported in more detail.

As always, let me know if you have any comments, questions or corrections:

<center>
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">I wrote about ContextThemeWrapper on Android, with a short example showing how it can be used with alert dialogs.<br><br>Thanks <a href="https://twitter.com/crafty?ref_src=twsrc%5Etfw">@crafty</a> for the 👀!<a href="https://t.co/Ew2ewfmD6b">https://t.co/Ew2ewfmD6b</a> <a href="https://twitter.com/hashtag/androiddev?src=hash&amp;ref_src=twsrc%5Etfw">#androiddev</a> <a href="https://twitter.com/hashtag/gde?src=hash&amp;ref_src=twsrc%5Etfw">#gde</a></p>&mdash; a-ta-ul ✏️ (@ataulm) <a href="https://twitter.com/ataulm/status/1196919491646509057?ref_src=twsrc%5Etfw">November 19, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</center>

_Thanks [Nick](https://twitter.com/crafty) for the review._


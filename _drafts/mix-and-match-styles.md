---
title: Mix and Match Styles on Android
excerpt: Let’s see how we can leverage a custom view inflater to implement (limited) support for mix and match styles. We’ll also revisit `materialThemeOverlay` to understand why we needed to add explicit support for this attribute in our custom views.
---

Themes and styles on Android can extend from another style, overriding or specifying additional attributes. It’s a single inheritance model where a style can’t have two parents.

TK image showing the boxes extending only one style at a time. 

Some other frameworks _do_ allow this, like [mixins in Sass](https://sass-lang.com/documentation/at-rules/mixin). The question is, how can we do this on Android?

### materialThemeOverlay using Context Theme Wrapper

Let's first recap what we know about `android:theme` and `materialThemeOverlay`.

We can use the `android:theme` attribute on a view directly or in a style resource that’s set using the view’s `style` attribute. `android:theme` support is afforded by the layout inflater (`AppCompatViewInflater`) which works by wrapping the context using `ContextThemeWrapper` before it creates the view.

`android:theme` doesn't work from default style resources. Default style resources allow us to set the _default_ style for a view, without having to specify a style explicitly at the point of usage. Since the default style resource (`defStyleRes` or `defStyleAttr`) is specified in the view's constructor, it's not accessible to the layout inflater, and therefore isn't able to check it for `android:theme`.

This is why `materialThemeOverlay` was added. In the [first post from this series]({% post_url 2019-10-14-using-material-theme-overlay %}), we saw how to use `materialThemeOverlay` by wrapping the context given to us in the constructor using a `ContextThemeWrapper`.

We use a convenience function, `MaterialThemeOverlay.wrap(context...)` ([available from 1.2.0](https://github.com/material-components/material-components-android/commit/115313c0c05d6fa404cbcdffa256877df8e86606)), to do this for us:

```kotlin
class MyCustomView(ctx: Context, attrs: AttributeSet) : View(
  MaterialThemeOverlay.wrap(ctx, attrs, R.attr.myCustomViewStyle, 0),
  attrs,
  R.attr.myCustomViewStyle
)
```

This function takes `defStyleAttr` and `defStyleRes` as parameters, so it's able to check the default style resource for `materialThemeOverlay` and wraps the context that's passed with `ContextThemeWrapper` if `materialThemeOverlay` was specified somewhere. There's two reasons a new attribute was introduced (instead of reusing `android:theme`):

1. People might have expected `android:theme` to work for any view, but this wouldn't be true. A custom attribute (no `android:` namespace) sets a clearer expectation that a subset of views will support it.
2. If `android:theme` was set in a default style resource, then it could be _replaced_ by `android:theme` usages in a `style` resource or directly on the view (read more about [how Android resolves view attributes here](TK link to blog)). This is expected behaviour, but using a different attribute means that multiple theme overlays can be applied.

One final consideration the function makes is to reapply the `android:theme` overlay after it's applied the `materialThemeOverlay` ones so that attributes defined in `android:theme` override ones in `materialThemeOverlay`.

### Mixins with multiple theme overlays

We can go deeper. If `materialThemeOverlay` affords an additional theme overlay, then we could just as easily layer our own theme overlays to support a mixins-ish behavior.

Let's say we've got the following mixins:

```xml
<style name="Mixin.Demo.LargeText" parent="">
    <item name="android:textSize">24sp</item>
</style>

<style name="Mixin.Demo.BlueText" parent="">
    <item name="android:textColor">@color/material_blue_500</item>
</style>
```

We could include them in our own styles like so:

```xml
<style name="Theme.Demo" parent="...">
    ...
    <item name="textViewStyle">@style/Widget.Demo.TextView</item>
    <item name="materialButtonStyle">@style/Widget.Demo.Button</item>
</style>

<style name="Widget.Demo.TextView" parent="Widget.MaterialComponents.TextView">
    <item name="include">@style/Mixin.Demo.LargeText</item>
    <item name="include2">@style/Mixin.Demo.BlueText</item>
</style>

<style name="Widget.Demo.Button" parent="Widget.MaterialComponents.Button">
    <item name="include">@style/Mixin.Demo.LargeText</item>
</style>
```

There's two ways to do this. If we're not fussed about supporting default style resources, we can do it in the layout inflater, otherwise we should follow the `materialThemeOverlay` approach with a wrapping function. In both cases, we'll define a few custom attributes in `res/main/values/attrs.xml`:

```xml
<declare-styleable name="View">
    <attr name="include" format="reference" />
    <attr name="include2" format="reference" />
    <attr name="include3" format="reference" />
    <attr name="include4" format="reference" />
</declare-styleable>
```

### Extending the layout inflater

We want to override the 8-param `createView()` from `AppCompatViewInflater`:

```kotlin
class MixinsViewInflater : MaterialComponentsViewInflater() {

    override fun createView(
        parent: View, name: String, context: Context,
        attrs: AttributeSet, inheritContext: Boolean,
        readAndroidTheme: Boolean, readAppTheme: Boolean, wrapContext: Boolean
    ) : View {
        return super.createView(
            parent, name, context.wrapWithIncludes(attrs),
            attrs, inheritContext,
            readAndroidTheme, readAppTheme, wrapContext
        )
    }
}
```

Unfortunately it's `final`. Instead, we can override the 3-param `createView()` (which is called in the default case, when inflating a view that won't be replaced, e.g. `Button` for `AppCompatButton`).

The 8-param `createView()` will call `createViewFromTag()` if the 3-param `createView()` returns null. So let's just copy and paste `createViewFromTag()` into our `createView()` so that it _doesn't_ return null.

This will allow us to create the view with the `ContextThemeWrapper`-wrapped context instead of whatever it was going to use. We also need to override all the functions that are called to replace views, otherwise these won't include our wrapped context.

```kotlin
class MixinsViewInflater : MaterialComponentsViewInflater() {

    // some constants used by `createViewWithTag` copied wholesale
    private val sConstructorSignature = arrayOf(Context::class.java, AttributeSet::class.java)
    private val sClassPrefixList = arrayOf("android.widget.", "android.view.", "android.webkit.")
    private val sConstructorMap = ArrayMap<String, Constructor<out View>>()
    private val mConstructorArgs = arrayOfNulls<Any>(2)

    override fun createView(ctx: Context, name: String, attrs: AttributeSet): View {
        val context = ctx.wrapWithIncludes(attrs)
        // ... the rest is as copied from `createViewWithTag`
    }

    override fun createTextView(context: Context, attrs: AttributeSet): AppCompatTextView {
        return super.createTextView(context.wrapWithIncludes(attrs), attrs)
    }

    override fun createImageView(context: Context, attrs: AttributeSet): AppCompatImageView {
        return super.createImageView(context.wrapWithIncludes(attrs), attrs)
    }

    // ... the other overrides

    // copied because createViewWithTag uses it
    @Throws(ClassNotFoundException::class, InflateException::class)
    private fun createViewByPrefix(context: Context, name: String, prefix: String?): View? {
        // ...
    }
}
```

Our `wrapWithIncludes(...)` function will read our custom attributes and repeatedly wrap the context:

```kotlin
fun Context.wrapWithIncludes(attrs: AttributeSet): Context {
        val typedArray = obtainStyledAttributes(attrs, R.styleable.View, 0, 0)
        val include = typedArray.getResourceId(R.styleable.View_include, 0)
        val include2 = typedArray.getResourceId(R.styleable.View_include2, 0)
        val include3 = typedArray.getResourceId(R.styleable.View_include3, 0)
        val include4 = typedArray.getResourceId(R.styleable.View_include4, 0)
        typedArray.recycle()

        return wrapThemeIfNecessary(include)
                .wrapThemeIfNecessary(include2)
                .wrapThemeIfNecessary(include3)
                .wrapThemeIfNecessary(include4)
    }

private fun Context.wrapThemeIfNecessary(@StyleRes themeRes: Int) =
        if (themeRes != 0 && (this !is ContextThemeWrapper || this.themeResId != themeRes)) {
            // If the context isn't a ContextThemeWrapper, or it is but does not have
            // the same theme as we need, wrap it in a new wrapper
            ContextThemeWrapper(this, themeRes)
        } else this
```












### Is this useful?

There isn't much value in fighting against the system to support mixins like this but it was fun to try. In addition, this approach has the same limitation as `materialThemeOverlay` in that it'll require explicit support in your views to work in default style resources.

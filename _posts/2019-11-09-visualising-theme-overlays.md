---
title: Visualising Theme Overlays
featured_image: /images/visualising-theme-overlays/feature.png
excerpt: A theme is just a style, used to define theme attributes. Let's use Connect 4 to visualise how Android theme overlays are applied, and the effect on attribute resolution.
---

![](/images/visualising-theme-overlays/feature.png)

A theme is a style resource that's associated with a `Context`. It's usually used to define theme attributes (though can be used to define view attributes), for example, `colorPrimary`.

`ContextThemeWrapper` is an implementation of `ContextWrapper` which lets us specify a theme. Since `ContextThemeWrapper` must have a "base" context (which will have its own theme), we can use this class to create theme overlays. `Activity` extends `ContextThemeWrapper` which is why an activity theme will take precedence over the application (base) theme.

When there are multiple view overlays, how do views resolve the attributes? We'll use the classic game, _Connect 4_ (also known as _Four in a Row_, _4 Gewinnt_, _Puissance 4_...) to help us understand.

Here, each overlay is represented by a different set of coloured counters, and each column represents a specific attribute that can be set in a theme:

![](/images/visualising-theme-overlays/theme-overlays.gif)

The red layer (on the bottom row) represents the application (base) theme. It hasn't defined a value for `fontFamily`.

![](/images/visualising-theme-overlays/app-dropped.png)

We can specify a theme for our activity using the `android:theme` attribute in the Android manifest. Since activity extends from `ContextThemeWrapper`, we have a field to store the activity theme, but we also keep the `mBase` field in the `ContextWrapper` class.

Even though there's a value for `fontFamily` now (for anything in that activity), we still have a reference to the original application theme; its attribute definitions aren't overridden.

![](/images/visualising-theme-overlays/activity-dropped.png)

We can do the same with theme overlays on views too. Here, the `colorPrimary` value is overridden in the context of this view:

![](/images/visualising-theme-overlays/view-theme-overlay.png)

Each attribute is resolved in isolation and is resolved using the outermost context. So, if we're looking for `colorPrimary` we look only at that column, and we'll start from the top (the latest overlay), so we'll use the value defined in the theme overlay from the view's `android:theme`.

The diagrams don't show this clearly (sorry!), but each theme overlay will be checked for each attribute, even if there's no values specified. I.e. when we're trying to resolve `colorOnBackground` the green and yellow layers would be checked first, before finding the value in the red (base theme) layer.

![](/images/visualising-theme-overlays/resolved.png)

Please [let me know if you found this post helpful](https://twitter.com/ataulm/status/1191494798097756165), or if you have any comments or questions (or corrections!).

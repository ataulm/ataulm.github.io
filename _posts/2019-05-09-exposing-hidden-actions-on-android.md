---
title: Exposing Hidden Actions on Android
# featured_image: /images/article-directory/featured-image.png
excerpt: Exposing “hidden” actions to accessibility services used to require providing a custom AccessibilityDelegate or using a third-party library like Novoda’s accessibilitools. No longer! There’s a new API dropping in the latest version of AppCompat (1.1.0) which includes a handy way to serve this information to the system with minimum effort.

---

Exposing “hidden” actions to accessibility services used to require providing a custom `AccessibilityDelegate` or using a third-party library like [Novoda’s accessibilitools](https://github.com/novoda/accessibilitools/).

No longer! There’s a new API dropping in the latest version of AppCompat (1.1.0) which includes a handy way to serve this information to the system with minimum effort.

## What is it?

The function adds additional information to a view which can be read by Android accessibility services. It associates a view with an “action”, where an action consists of a label and executable block of code.

Accessibility services, like TalkBack, can present these actions to users in a way that’s familiar to them. The GIF shows two actions, “watch” and “download”, which have been associated with a view, and how a user can access them with TalkBack:

![](/images/exposing-hidden-actions-on-android/custom-action-click.gif)

## How do I use it?

Use `ViewCompat` to set an action on a view, passing the label for the action, and the action to perform:

```kotlin
val watchLabel = getString(R.string.action_watch)
ViewCompat.addAccessibilityAction(itemView, watchLabel) { _, _ ->
    onWatchClicked()
}
```

We’ll repeat this for each action we need to associate:

```kotlin
val downloadLabel = getString(R.string.action_download)
ViewCompat.addAccessibilityAction(itemView, downloadLabel) { _, _ ->
    onDownloadClicked()
}
```

And that’s it! The `itemView` now has two actions, “watch” and “download”, which are available to accessibility services like TalkBack.

It works using an `AccessibilityDelegate` under the hood to add actions to `AccessibilityNodeInfo`. It uses view tags which allows it to add actions over multiple calls without passing them all at once, but we’re limited to 32 actions per view, which is more than enough for a single view.

## When should I use this?

This is useful when we have actions on a view that aren’t accessible with a simple click—e.g. a swipe-to-delete gesture.

See the “Single-action elements” section in [“What’s Next? A Practical Introduction to Accessibility on Android”](https://proandroiddev.com/whats-next-a-practical-introduction-to-accessibility-on-android-79c90d27e124) for more ideas on when this might be useful.

If you have any questions, comments or corrections, [let me know](https://twitter.com/ataulm)!

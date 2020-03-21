---
title: Can TalkBack Read Me A Story?
# featured_image: /images/article-directory/featured-image.png
excerpt: Someone asked me the other day how accessibility services behave if you use a single TextView with multiple spans, instead of multiple TextViews. I didn’t know, so I tried doing it.

---

Someone asked me the other day how accessibility services behave if you use a single textview with multiple spans, instead of multiple textviews. I didn’t know, so I tried doing it.

Let’s start with three regular textviews:

```xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical">

    <TextView
        android:id="@+id/beginning"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textStyle="bold" />

    <TextView
        android:id="@+id/middle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <TextView
        android:id="@+id/end"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textStyle="italic" />

</LinearLayout>
```

We set the styles differently, so each section is clear, then bound text to the views:

```kotlin
beginning.text = "A Short Story"
middle.text = "This is a short story"
end.text = "fin."
```

![An app showing three lines: “A Short Story”, “This is a short story” and “fin.”, each with a different visual style](/images/talkback-and-spans/short-story.png)

With TalkBack enabled, how will it sound? The gif below shows that it treats each TextView independently; a swipe-right “next” gesture is required to navigate to the next line, where the text is then read aloud.sho

![Gif showing TalkBack traversal between each line](/images/talkback-and-spans/short-story-talkback.gif)

Let’s try the same thing but with **one** textview, using spans to style the text.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_my)
    story.text = writeFormattedStory()
}

private fun writeFormattedStory(): CharSequence {
    return SpannableStringBuilder()
        .append(createWithSpans("A Short Story", StyleSpan(Typeface.BOLD)))
        .append("\n")
        .append("This is a short story")
        .append("\n")
        .append(createWithSpans("fin.", StyleSpan(Typeface.ITALIC)))
}

private fun createWithSpans(charSequence: CharSequence, vararg spans: Any): SpannableString {
    val spannableString = SpannableString(charSequence)
    for (span in spans) {
        spannableString.setSpan(span, 0, charSequence.length, Spannable.SPAN_EXCLUSIVE_EXCLUSIVE)
    }
    return spannableString
}
```

TalkBack will read the text out, pausing between each one (since there are line breaks separating them) — no need to swipe between each line.

If the user wants, they can change the granularity at which the text is read aloud, so instead of reading the entire view (default), it could read lines, words or even letters, requiring a “next” gesture to move to the next.

![The screen is selected and captions for TalkBack are displayed. Each of the three lines have been read aloud, with a pause between each](/images/talkback-and-spans/short-story-talkback-custom.png)

Depending on the span though, TalkBack will add additional auditory cues!

(This article was more interesting before TalkBack 6.0 was released…)

In [earlier versions of TalkBack](https://github.com/google/talkback/blob/12bdf2063e121a021f050c94cf5ebb2489c8af8a/src/main/java/com/android/talkback/FeedbackProcessingUtils.java#L242), the pitch would be adjusted to reflect the style span, and an earcon would be added (portmanteau of ear and icon — a sound that plays to help the user understand the context of what’s being read):

```java
/**
 * Handles the splitting of {@link URLSpan}s into multiple
 * {@link FeedbackFragment}s.
 *
 * @param fragment The fragment containing the spannable text to process.
 * @param span The individual {@link StyleSpan} that represents the span
 */
private static void handleStyleSpan(FeedbackFragment fragment, StyleSpan span) {
    final int style = span.getStyle();

    final int earconId;
    final float voicePitch;
    switch (style) {
        case Typeface.BOLD:
        case Typeface.BOLD_ITALIC:
            voicePitch = PITCH_CHANGE_BOLD;
            earconId = R.raw.bold;
            break;
        case Typeface.ITALIC:
            voicePitch = PITCH_CHANGE_ITALIC;
            earconId = R.raw.italic;
            break;
        default:
            return;
    }

    final Bundle speechParams = new Bundle(Bundle.EMPTY);
    speechParams.putFloat(SpeechController.SpeechParam.PITCH, voicePitch);
    fragment.setSpeechParams(speechParams);
    fragment.addEarcon(earconId);
}
```

How cool is that! I guess it wasn’t very useful or people didn’t notice though, since with TalkBack 6.0, they’ve dropped this functionality. Not for everything, mind. If the text contains any _clickable_ spans (e.g. URLSpan), then this is [still handled as a special case](https://github.com/google/talkback/blob/12aea503b7f21713065a65014031ef62307509ed/utils/src/main/java/output/FeedbackProcessingUtils.java#L238) — you can hear a faint pop just before the link text is read aloud:

```java
/**
 * Handles {@link FeedbackFragment} with {@link ClickableSpan} (including {@link URLSpan]}). Adds
 * earcon and pitch information to the fragment.
 *
 * @param fragment The fragment containing {@link ClickableSpan}.
 */
private static void handleClickableSpan(FeedbackFragment fragment) {
    final Bundle speechParams = new Bundle(Bundle.EMPTY);
    speechParams.putFloat(SpeechParam.PITCH, PITCH_CHANGE_HYPERLINK);
    fragment.setSpeechParams(speechParams);
    fragment.addEarcon(R.raw.hyperlink);
}
```

And how are these clickable spans accessed by TalkBack users? We already saw above that the entire textview is selected, so they can’t just tap on the links directly.

In my understanding, the only way for TalkBack users to access these links is to use the local context menu for the textview; the “Links” option will contain the hyperlinks that the user can then select from the list dialog.

![Gif shows similar text, but with two hyperlinks indicated by different color text and underline. The local context menu is used to navigate to the two links individually.](/images/talkback-and-spans/short-story-talkback-with-link.gif)

Accessing the local context menu might not be possible for all users, and even if it is — what a pain to have to go through those steps for links! Does that mean you shouldn’t use spans if you want to make an app accessible for TalkBack users?

No way, you can do what you like. We can provide alternative UX to make it more straight-forward by detecting the presence of a spoken-feedback accessibility service like TalkBack, or even a keyboard/switch-access device (I haven’t yet tested how these clickable spans would be used in those cases).

For this particular case, I would recommend the pattern as when we’re trying to develop single-action views, where we hide inline-actions (like “play” buttons or in this case hyperlinks), and instead have a single action on-tap, which opens a dialog listing all available actions for that view.

This is the approach taken in apps like Google+ and the [technique is described here](https://blog.novoda.com/alternative-interfaces/).

If you have any questions, comments or corrections, [let me know](https://twitter.com/ataulm)!

---
title: What’s Next? A Practical Introduction to Accessibility on Android
featured_image: /images/practical-accessibility/feature.png
excerpt: The past two years have seen a lot more interest for inclusive design in the Android community. This post is a hands-on, practical introduction to accessibility on Android.

---

The past two years have seen a lot more interest for inclusive design in the Android community. There’s been a lot of content (blogs and talks) introducing waves of designers and developers to accessibility.

![An accessibility champion, leaping through the air with a battle-axe and wooden shield](/images/practical-accessibility/champion.png)

Some of these people have since taken up an “accessibility mantle”, **championing** inclusive design within their teams and companies.

But while it’s useful to have an enthusiastic supporter or expert around, having only one person carrying the torch isn’t sustainable for a product — you need the whole team!

![A despondent butterfly catcher surrounded by butterflies, next to another who is catching them methodically](/images/practical-accessibility/team.png)

Maybe the reason for this poor uptake is because we moved too quickly, or focused on the wrong parts.

A lot of the talks and blog posts dish out great tips and best practices, but they can be overwhelming.

It means that people can come away from the experience feeling that accessibility is a “Good Thing To Do”, but find it difficult to start. This is especially hard if there is no context or framework in which to make changes.

On Twitter I asked, [what did people need?](https://twitter.com/ataulm/status/967764961375260677)

<center>
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">imagine you&#39;re in a position (💵 and ⌚️) to make your app accessible (vague word is vague).<br><br>what&#39;s stopping you?<br><br>would you prefer examples from scratch, base concepts, migration techniques, etc? Something else? RTs and replies appreciated 🙌<a href="https://twitter.com/hashtag/androiddev?src=hash&amp;ref_src=twsrc%5Etfw">#androiddev</a></p>&mdash; ataúl ✏️ (@ataulm) <a href="https://twitter.com/ataulm/status/967764961375260677?ref_src=twsrc%5Etfw">February 25, 2018</a></blockquote>
</center>

![Three helpful folk from the community, giving feedback](/images/practical-accessibility/community1.png)

The responses (public and private) were varied. I had questions such as “how can we integrate accessibility checks into our build process?” or “how should we change the UX to adapt for different users?”.

The majority of the responses, however, were about getting started!

![Another three helpful folk from the community also giving feedback](/images/practical-accessibility/community2.png)

- “I’d love to spend time on making accessibility better, but where do I even start? What’s the most important thing to do?”
- “How can we identify different user personas?” “I’d love some guidelines and a comprehensive list of all the situations (not only blind or color blind)”
- “Working through an example app that has a few intricacies to it”

There were also questions from the business side:

- “Budget is the main constraint… special provisions are often out of scope as time does not permit them”
- “Ideas on how to push companies to make it happen & understand how important it is”

## Getting started

![](/images/practical-accessibility/chalkboard.png)

Let’s re-frame the context of what it means to make an app accessible. There are two things that every app does:

- convey information to the user
- expose actions for the user to perform

Our job, as developers and designers, is to facilitate these two things for as many users as possible — and that’s all accessibility is.

## A real-world example

Let’s use YouTube as a non-trivial case study. It works because it includes a lot of the widgets and design-patterns we encounter day-to-day, as well as other components (like video) that we might not have experience using in our apps.

![](/images/practical-accessibility/youtube.png)

We’ll highlight the things it does well, and also look at opportunities where we can approach things differently.

A lot of the following content might seem simple in theory, but that’s good. The idea is to fill gaps not only in our understanding but also our designs and implementations. When you’ve tackled the issues presented here, it’s your turn to say, “what’s next?”

We’ll cover two topics in this article, but for me, addressing these issues will make the biggest difference for your users.

## Text

There’s only one thing that text needs to be — readable. This section will focus on how to prioritize the readability of your text, i.e. make it accessible to your users.

One of the hardest things we have to do is design for non-static content. For most of us, the content we display to the user is delivered to us from a remote server.

Even for text that we know at design time, we don’t know how translations will affect the length. **To keep our text readable, we need to make sure it’s never cut off.**

![](/images/practical-accessibility/magic.png)

YouTube displays information about videos sourced entirely from uploaders. It uses several techniques to handle dynamic content!

Create fully responsive layouts wherever possible. On the YouTube homepage, each item’s width matches the device width, but the height is unconstrained.

![Landscape and portrait view of the YouTube homepage on Android](/images/practical-accessibility/youtube-homepage.png)

Because the list is vertically scrollable, items can grow (vertically) without restriction. Video titles can be as long or short as they like, with text wrapping onto new lines.

**Double-check views that have a fixed-size.** On the video page, there’s a scrollable list of related content. Here, a fixed height means more items can fit on screen.

But look, one of the titles is no longer fully readable!

![Left: list of related videos on video page with video title ellipsized on the first item, right: video page for that item showing the full title](/images/practical-accessibility/youtube-ellipsis.png)

It’s OK! **Ellipses can serve as a subtle call-to-action.** Here, they let the user know that clicking on the item will take them somewhere they can read the video title in its entirety.

**Split content into multiple parts.** Users can write very long descriptions for videos. By default, the app only shows the title, and some stats. Clicking on the title or the arrow will expand the view to show the description and some of the stats in more detail:

![Left: video metadata collapsed, middle: expanded, right: expanded, highlighting the views, subscribers and description](/images/practical-accessibility/youtube-expand.png)

Some of the channels are really popular, with millions of subscribers. In the expanded form, this means that the text might not fit:

![Left: collapsed form shows “4.9M subscribers”, right: expanded form shows “4,981,117 subscrib…”](/images/practical-accessibility/subs-ellipsis.png)

In this case, we could choose to wrap the text onto a new line, increasing the height of the container, instead of ellipsizing the label.

So now we’ve handled the cases where text might be cut off. Great! What else do we have to do in our effort to keep text readable?

**Test your app at various font scales.** The [user can change the font scale](https://support.google.com/accessibility/android/answer/6006972) at the system level and our apps should support this by using [scaleable pixels](https://issuetracker.google.com/issues/73425984) to specify text size. Anything that used “sp” units to define size will scale relatively:

![Side by side of YouTube homepage, with normal text size on left, and enlarged text size on right](/images/practical-accessibility/youtube-large-text.png)

If you have text in your images, then those images should scale as well (or avoid putting text in images!). Consider using scaleable pixels to define dimensions for icons too. What do you think the effects would be?

Users aren’t able to choose arbitrary values for the text size. They must select from “small”, “normal”, “large” and “huge”, which ranges from 0.8–1.3x the normal size. This means you can limit your testing to these extreme values.

[Novoda has a JUnit test rule](https://github.com/novoda/espresso-support/blob/4099033bef475d601507c5338b8e23e98a3e3d40/core/src/main/java/com/novoda/espresso/FontScaleRules.java) that can help you set the font scale on devices during your Espresso test runs, and reset them afterwards. The system font scale will be set to huge before VideoActivity is launched:

```kotlin
// FontScaleRules available from "novoda/espresso-support"
class VideoActivityTest {

    val activityRule = ActivityTestRule(VideoActivity::class.java)

    @Rule
    val ruleChain = RuleChain.outerRule(FontScaleRules.hugeFontScaleTestRule())
            .around(activityRule)
}
```

After the test runs, `VideoActivity` will be torn down and then the font scale will be reset to its original value, ready for the next test.

The final part to making your text readable is being color considerate. There’s two requirements to this:

- ensure information is conveyed not only by color
- ensure sufficient contrast between text and its background

The bottom navigation bar in YouTube, for instance, indicates the selected page by coloring it red. Let’s compare what it looks like for users with a specific [type of color blindness called protanopia](http://www.color-blindness.com/types-of-color-blindness/):

![active icon is red, right: active icon is black, indistinguishable from other](/images/practical-accessibility/youtube-nav-red.png)

![active icon is black, indistinguishable from other](/images/practical-accessibility/youtube-nav-black.png)

Even something subtle, like changing the font weight can be useful as a secondary information vector.

For users with protanopia, the active icon would still be distinct from the others:

![Navigation bar with home icon as red and bolder than the other icons](/images/practical-accessibility/youtube-nav-bold.png)

**Test your color palettes** — there are handy apps you can use on your computer for this ([Sim Daltonism for OSX](https://itunes.apple.com/au/app/sim-daltonism/id693112260?mt=12)), and you can even test directly on your device with the Simulate Color Space feature under Developer Options.

**Color contrast is the “difference” between two colors.** For us, we usually mean the color of text and the color of its background, but it can also apply to iconography.

Although the idea is simple, it’s not always straightforward because the colors may be generated at runtime with utilities like [Palette](https://developer.android.com/reference/android/support/v7/graphics/Palette.html) or the background may be an image or video.

Playing video | with video overlay
---|---
![YouTube video playing](/images/practical-accessibility/youtube-video.png) | ![controls displayed on a darkened video overlay](/images/practical-accessibility/youtube-video-overlay.png)

YouTube uses a darkened background when displaying controls on top of a video. This means that the white text is always readable and white icons are always distinguishable, regardless of the video color.

## Accessibility Services

A structured guide to accessibility on Android would be incomplete without mentioning accessibility services!

An [accessibility service](https://developer.android.com/reference/android/accessibilityservice/AccessibilityService.html) is a software-based assistive technology — it helps the user interact with their device, and it still does the two things we mentioned at the start:

- helps the app convey information to the user
- helps the user perform actions in a given app
As app developers, we don’t need to do much to make our apps compliant with accessibility services — the system does most of the heavy lifting for us.

However, by putting a little effort in, we can improve the usability of our apps for users of these services.

### Google TalkBack

![](/images/practical-accessibility/talkback.png)

Google TalkBack is probably the most well known accessibility service. It’s used by blind people or users with severe visual impairments.

TalkBack acts as a screenreader and, via gestures, provides users with affordances to navigate between, and interact with, elements on screen.

With TalkBack, users navigate through elements on screen sequentially, using gestures like “swipe right” to go to the next element, or “left” to go to the previous.

![YouTube app bar](/images/practical-accessibility/youtube-appbar.gif)

Double-tapping anywhere on the screen will send a click event to whatever’s focused.

The usual way to navigate an unfamiliar app is to use these gestures to get through the page one item at a time. However, the user can choose to tap anywhere on the screen to move the focus directly to that view.

![Animation showing TalkBack traversing through views on the screen](/images/practical-accessibility/youtube-talkback.gif)

This can be useful on screens with long lists — by default, using the “next” gesture will cause the list to scroll, which means it’s easy to get trapped in a long list.

Instead, with the Touch-to-Explore setting, the user can jump to the bottom navigation or the app bar without having to go through every list item.

### Content descriptions

When I was collecting feedback, someone asked “what makes a good content description?” Since TalkBack is a spoken feedback accessibility service, it reads the content descriptions of views to let the user know what’s on screen.

Some View types have inherent descriptions; anything extending TextView will read the text aloud. For images and other views, you need to set the content description yourself. Say what it is, and where applicable, the state.

```xml
<Button
    android:id="@+id/likeButton"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Like" />

<ImageView
    android:id="@+id/likeBadge"
    android:src="@drawable/ic_heart_filled"
    android:layout_width="24dp"
    android:layout_height="24dp"
    android:contentDescription="Liked" />
```

Remember to describe what the image represents, not the image itself. For the app bar, it’s enough to say “YouTube” and “Search” rather than “YouTube logo” and “magnifying glass”, because that better describes the purpose of these images.

**Specify explicit content descriptions for ViewGroups too.** It’s not just individual Views that need content descriptions. Setting the description on a complex ViewGroup allows you to control what TalkBack is going to say as well as the order.

![Comment thread in YouTube, one comment is focused in TalkBack](/images/practical-accessibility/youtube-comment.png)

If you don’t, TalkBack will concatenate the content descriptions of all the children, which can be dangerous as the View changes through app and feature updates.

So we said that a good content description should describe what a View is, not what it does. But then how does the user know whether an element is actionable or not?

Previously, the recommended approach was to append the word “button” to the content description. This lets the user know that the element is clickable.

Instead, **add _usage hints_ to actionable views**. Usage hints are automatically added to clickable Views — after the content description is read, there’s a brief pause, followed by the hint.

![TalkBack captions read: “Double tap to play video”](/images/practical-accessibility/youtube-usage-hints.png)

By default, it says “Double tap to activate”. However, from Lollipop and above, we can customize the action. So how do we do that?

[AccessibilityDelegates](https://developer.android.com/reference/android/support/v4/view/AccessibilityDelegateCompat) let us override some of the methods in View that relate to accessibility.

We can add additional metadata about the View to [AccessibilityNodeInfo](https://developer.android.com/reference/android/support/v4/view/accessibility/AccessibilityNodeInfoCompat) objects, as shown below, specifying a custom label for a click action.

```kotlin
ViewCompat.setAccessibilityDelegate(itemView, object: AccessibilityDelegateCompat() {
    override fun onInitializeAccessibilityNodeInfo(host: View, info: AccessibilityNodeInfoCompat) {
        super.onInitializeAccessibilityNodeInfo(host, info)
        val clickActionId = AccessibilityNodeInfoCompat.ACTION_CLICK
        info.addAction(AccessibilityActionCompat(clickActionId, "play video"))
    }
})
```

If you only need to set a usage hint, you can add a [Kotlin extension function](https://github.com/android/android-ktx/pull/483/) to simplify this to one line, writing only:

```kotlin
itemView.setUsageHint("play video")
```

That’s pretty much it if you want a base level of support for TalkBack: explicitly-set content descriptions and usage hints. But let’s raise the bar.

### Single-action elements

Linear navigation means going through a feed can be tedious, since TalkBack will focus on all the inline actions as well as the item itself.

What we can do is [detect whether an accessibility service like TalkBack is enabled](https://github.com/novoda/accessibilitools), and provide alternative behavior, like removing any inline actions:

![Animation showing navigation between items in TalkBack where only the whole item gets focused, no inline actions](/images/practical-accessibility/youtube-single-action.gif)

Instead of three clickable areas per item, we have one. This means you only need a single gesture to navigate between elements.

But hang on a minute! We can’t just get rid of actions and reduce functionality under the guise of simplicity!

Instead, when a user clicks on an item, we’ll display all the actions for that video in a dialog.

![Left: item selected by TalkBack, right: dialog showing available actions for that selected item](/images/practical-accessibility/youtube-dialog.png)

Doing this across the app wherever we’re looking to simplify ViewGroups will lead to a consistent user experience for TalkBack users.

To expose these actions directly to the accessibility service, we can override two methods from the same AccessibilityDelegate that we used earlier for usage hints:

```kotlin
ViewCompat.setAccessibilityDelegate(itemView, object: AccessibilityDelegateCompat() {
    override fun onInitializeAccessibilityNodeInfo(host: View, info: AccessibilityNodeInfoCompat) {
        super.onInitializeAccessibilityNodeInfo(host, info)
        info.addAction(AccessibilityActionCompat(R.id.play_video, "play video"))
    }

    override fun performAccessibilityAction(host: View?, action: Int, args: Bundle?): Boolean {
        if (action == R.id.play_video) {
            // TODO: take action
            return true;
        }
        return super.performAccessibilityAction(host, action, args)
    }
})
```

These allow power-users to trigger actions for a given View using other gestures.

To make it easier to implement this and the dialog functionality without duplicating code, you can use [accessibilitools](https://github.com/novoda/accessibilitools).

```kotlin
// github.com/novoda/accessibilitools
val actionsMenuInflater = ActionsMenuInflater.from(getContext())
val listener: MenuItem.OnMenuItemClickListener = // ...
val menu = actionsMenuInflater.inflate(R.menu.actions, listener)

val delegate = ActionsMenuAccessibilityDelegate(menu, listener)
ViewCompat.setAccessibilityDelegate(itemView, delegate)
```

This library allows you to specify actions as a menu resource and callbacks with a MenuItemClickListener, and provides the custom AccessibilityDelegate you’ll need.

A similar technique can be used for sliding tab strips or bottom navigation, allowing the user to access navigation controls when necessary, but without requiring them to traverse each item individually.

_There’s an updated way to do this. See [“Exposing hidden actions on Android”]({% post_url 2019-05-09-exposing-hidden-actions-on-android%})._

### Add custom actions

If an accessibility service is enabled, YouTube doesn’t auto-dismiss the media controls on a video after a few seconds. Instead, it keeps them displayed indefinitely and adds an explicit dismiss button to the overlay.

![video overlay with controls](/images/practical-accessibility/youtube-overlay-controls.png)
![same overlay but with additional controls including dismiss button and skip/rewind 10 seconds buttons](/images/practical-accessibility/youtube-overlay-controls-extra.png)

In addition, since SeekBar controls are fiddly to use with TalkBack, there’s additional rewind and forward buttons available.

If it makes sense, consider adding custom actions that are available for users of accessibility services in place of (or in addition to) other affordances.

![](/images/practical-accessibility/yellow-community.png)

At the end of the day, each of the individual considerations aren’t difficult to implement. The difficulty lies in keeping them consistent in your app, between screens and between versions.

For me, the next step is working on how to bake this into a process that spans design, development and QA. I hope these examples help give you actionable stages you can work towards.

If you have any questions, comments or corrections, [let me know](https://twitter.com/ataulm)!

_Thanks to [Zarah](https://twitter.com/zarahjutz), [Maria](https://twitter.com/marianeum), [Pavlos](https://twitter.com/tpavlos), [Ben](https://twitter.com/benlikestocode) and [Raúl](https://twitter.com/RaulHernandezL) for feedback on this post!_

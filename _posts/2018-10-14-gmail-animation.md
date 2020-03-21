---
title: Implementing the Gmail for Android selected email animation
# featured_image: /images/article-directory/featured-image.png
excerpt: This template includes some of things that you always forget when you’re writing a blog post. With this post, you’ll be able to copy and paste stuff. Be sure to remove the comments before publishing!

---

I really like the way that Gmail for Android shows the checked state for selected emails.

![The avatar spins 180 degrees, revealing a checkmark on the other side](/images/gmail/gmail-animation.gif)

Why is this good? There are **three visual affordances** which make it very clear which emails are selected:

- the avatar is replaced by a checkmark
- the email’s background changes color
- the number of selected emails is displayed in the app bar

Why is it **COOL**? Look what happens when an email is selected and deselected quickly. The avatar animation is shown in reverse from the position it was at:

![](/images/gmail/gmail-reverse.gif)

I have **five** requirements:

1. Display a list with items already selected (no animation).
2. Persist the selected state in the Presenter/ViewModel, not in the View.
3. Animate transitions to the checked and the not-checked states.
4. Report the transition to the checked/not-checked states back to the Presenter/ViewModel immediately. I don’t want to wait for the animations to complete, so other parts of the UI can update if necessary (like the app bar)
5. Handle spammy clicks as gracefully as Gmail — no jumping between the checked/not-checked styles, everything should be animated

I used the [`Rotate3dAnimation` class](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/animation/Rotate3dAnimation.java#) from the Android samples to do the swivel itself. The main thing I changed from the sample was adding the `matrix.preScale(-1f, 1f, centerX, 0f)` so that the “back” of the view wasn’t reversed.

I didn’t manage to get the fifth requirement completed — luckily, it doesn’t look too bad played at the regular animation speed:

![](/images/gmail/implemented.gif)

See the [full solution here](https://github.com/ataulm/android-skeleton/blob/fc3b88d4e30f73b5702a4efa2abc61e3fcc009f2/app/src/main/java/com/example/SwivelCheckView.kt).

How would you have done this? Is the reverse easier than I was thinking or are there any tricks I’m missing out on? Let me know [on Twitter](https://twitter.com/ataulm)!

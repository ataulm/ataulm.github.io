---
title: What’s Next? Watching Netflix with Help from Accessibility Services
# featured_image: /images/article-directory/featured-image.png
excerpt: When you start learning about accessibility on Android, it won’t be long before you come across Accessibility Services. These are special apps on Android that help the user interact with their device where some help the user consume information and others help perform actions.

---

When you start learning about accessibility on Android, it won’t be long before you come across [Accessibility Services](https://developer.android.com/reference/android/accessibilityservice/AccessibilityService.html).

These are special apps on Android that help the user interact with their device where some help the user consume information and others help perform actions.

We touched on the topic in a [previous post]({% post_url 2018-04-22-practical-accessibility %}). We learned:

>As app developers, we don’t need to do much to make our apps compliant with accessibility services — the system does most of the heavy lifting for us.

Have you heard of TalkBack, Switch Access or Voice Access? These aren’t designed to make specific apps accessible. Instead, they’re meant to help users interact with any app on their device.

We **don’t need to create an accessibility service to make our apps accessible** — we should make sure that we’re exposing enough information about our UI to the system, so that accessibility services can use that to serve the user.

But when _do_ we need one and are they difficult to write?

## Identify a problem

I watch Netflix on my Android TV with the remote app on my phone because the hardware remote stopped working.

This is fine, except when you have to click “Skip Intro” at the start of every episode or “Continue Watching” every three episodes.

![left: Nexus Player remote, right: Android TV app](/images/accessibility-service/remote.png)

On the hardware remote, this would only be one click. But on the software remote, you have to unlock the device, open the app from the notification shade and then click. The button auto-hides after a few moments, so you’ll have to be quick!

I wish I could just get it to auto-click for me.

## Building blocks

Enter Android’s accessibility services! These tiny heroes can do the job because they’re able to **observe changes to the screen content and take action on the user’s behalf**.

There are three things we need to do in order to get a running accessibility service:

- Create a class that extends from AccessibilityService
- Configure the service’s properties, either in code or XML
- Register the service in the Android Manifest

We’ll do this with [Skipper](https://github.com/ataulm/skipper), an accessibility service that you can rely on to navigate you through the treacherous waters of uh… watching Netflix and YouTube.

## Skipper

With Skipper, you’ll be able to sit back and relax as Netflix throws up the “Are you still watching?” prompt.

No longer will you have to pounce on “Skip ad” after waiting five seconds for it to appear in YouTube.

Let Skipper sail this boat.

In terms of UI, we want the user to be able to configure which words should be clicked. And we should also limit these to specific apps to mitigate unintentional clicks.

We’ll not go through how these work because it’s too specific to this app — check out the repository if you want to see the inner workings.

Instead, let’s start with the service itself.

## SkipperAccessibilityService.kt

There’s two methods we need to override from AccessibilityService:

```kotlin
class SkipperAccessibilityService : AccessibilityService() {

    override fun onInterrupt() {
        // no op - this has no feedback so there'll be nothing to interrupt
    }

    override fun onAccessibilityEvent(event: AccessibilityEvent) {
        // TODO: scan view hierarchy for clickable words
    }
}
```

We can ignore `onInterrupt()` because our service doesn’t output any feedback, unlike TalkBack which has spoken feedback, so there’s nothing to interrupt. If we did, this is where we should halt any output.

The other one is more interesting. Here, we get an `AccessibilityEvent` which comes from an `AccessibilityNodeInfo` which can be used to represent views in the hierarchy.

The following snippet prints the `text` property of each of the leaves in the view hierarchy — this means buttons, like “Skip Intro”!

```kotlin
class ViewDumpAccessibilityService : AccessibilityService() {
    ...
    
    override fun onAccessibilityEvent(event: AccessibilityEvent) {
        Log.d("!!!", "...")
        printLeaves(event.source)
        Log.d("!!!", "...")
    }

    private fun printLeaves(info: AccessibilityNodeInfo?) {
        if (childCount(info) == 0 && info?.text != null) {
            Log.d("!!!", info.text.toString())
        } else {
            printLeavesForEachChildIn(info)
        }
    }

    private fun printLeavesForEachChildIn(info: AccessibilityNodeInfo?) {
        for (i in 0 until childCount(info)) {
            printLeaves(info?.getChild(i))
        }
    }

    private fun childCount(info: AccessibilityNodeInfo?) = info?.childCount ?: 0
}
```

Printing all the leaves isn’t particularly useful though. We need to search for a specific one, and then click on it (if we can find it).

In `SkipperAccessibilityService`, we’ve loaded clickable words in map (`Map<AppPackageName, List<ClickableWord>>`).

That way, whenever we receive an `AccessibilityEvent` we can check which package it came from, and load the associated `List<ClickableWord>`.

Then, for each one of these words, we search for it, and try to click by performing an `ACTION_CLICK`. If we’re successful, we return. If not, we’ll move onto the next word.

```kotlin
class SkipperAccessibilityService : AccessibilityService() {

    ...
    
    override fun onAccessibilityEvent(event: AccessibilityEvent) {
        val clickableWords = appsToWordsMap[AppPackageName(event.packageName.toString())].orEmpty()
        
        clickableWords.forEach {
            val matchingNodes = event.source?.findAccessibilityNodeInfosByText(it.word).orEmpty()
            matchingNodes.forEach { node ->
                // we want at most one successful click from any of the nodes, matching any of the words
                if (node.clickClosestAncestor()) {
                    return
                }
            }
        }
    }

    /**
     * @return true if something was clicked
     */
    private fun AccessibilityNodeInfo?.clickClosestAncestor(): Boolean {
        if (this == null) {
            return false
        }

        if (hasClickAction()) {
            performAction(AccessibilityNodeInfo.ACTION_CLICK)
            return true
        } else {
            return parent.clickClosestAncestor()
        }
    }

    private fun AccessibilityNodeInfo.hasClickAction(): Boolean {
        actionList?.forEach({
            if (it.id == AccessibilityNodeInfo.ACTION_CLICK) {
                return true
            }
        })
        return false
    }
}
```

`AccessibilityNodeInfo?.clickClosestAncestor()` allows us to handle the cases where the text we’re looking for _isn’t_ clickable. For example, “Skip Intro” could be a non-clickable label, inside a clickable `LinearLayout`.

But where do these `AccessibilityEvent` objects come from? And can we filter them so we only get a subset? (Yes.)

## Accessibility Service Properties

[Accessibility services can be configured](https://developer.android.com/guide/topics/ui/accessibility/services#service-config) programmatically by creating and setting the `AccessibilityServiceInfo`, but we’ll do it with an XML file (`res/xml/skipper_config.xml`) since we can’t set all properties at runtime.

For example, `android:canRetrieveWindowContent` needs to be set in XML because it gives the service access to the View hierarchy, potentially exposing user-sensitive information. Setting it in XML means the system will warn the user when they enable the service.

Here’s how it looks for Skipper:

```xml
<?xml version="1.0" encoding="utf-8"?>
<accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:accessibilityEventTypes="typeWindowContentChanged"
    android:accessibilityFeedbackType="feedbackGeneric"
    android:accessibilityFlags="flagIncludeNotImportantViews|flagRetrieveInteractiveWindows"
    android:canRetrieveWindowContent="true"
    android:notificationTimeout="500"
    android:settingsActivity="com.ataulm.skipper.settings.SkipperSettingsActivity" />
```

We want to be informed whenever the content in the window changes — like, when a “Skip Intro” button appears. We can set flags too that complement this setting.

`flagIncludeNotImportantViews` will give us events from views that are marked with `android:importantForAccessibility="no"`. (I included this to highlight that accessibility services still have access to these Views, so don’t mark things as “not important” thinking it’ll improve the app’s security.)

`flagRetrieveInteractiveWindows` gives us content in dialogs too.

`android:settingsActivity` should be the fully qualified component name for the settings Activity. For Skipper, this is the screen where the user can add `ClickableWords` to a particular app.

`android:accessibilityFeedbackType` lets us specify what kind of output we provide to the user. Since there’s no “feedbackNone” (Skipper doesn’t provide feedback), we’ll use the generic one.

## Registering the service

All services must be registered and accessibility services are no exception.

```xml
    ...
    <service
        android:name="com.ataulm.skipper.SkipperAccessibilityService"
        android:label="@string/app_name"
        android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">

        <intent-filter>
            <action android:name="android.accessibilityservice.AccessibilityService" />
        </intent-filter>

        <meta-data
            android:name="android.accessibilityservice"
            android:resource="@xml/skipper_config" />
    </service>
</application>
```

This is where you can associate the configuration XML resource with the service.

---

It works! On the version of Netflix that runs on smartphones and tablets.

Unfortunately, it seems the Android TV version (a different APK) doesn’t use the Android View system so there’s none of this lovely metadata that the accessibility service can read.

(This also means that TalkBack and Switch Access won’t function on Netflix’s Android TV app.)

## OK, what’s next?

There’s more work to do with accessibility services and more questions to answer:

- what happens if you have multiple services that specify the same `feedbackType`? The docs say that only the first one will receive events.
- drawing on top of other apps (like TalkBack’s accessibility focus rectangles)
- interactive services, like Select to Speak, where you click on a View to have it read aloud
- integration with the accessibility shortcut introduced in Android 8.0

This post was more like “what’s _not_ next?”, since we’ve established that developing accessibility services is unlikely to improve your app.

Writing this service for Android TV does give us a good starting point for thinking about developing apps with support for non-touch input!

So keep an eye on the [Skipper repository on GitHub](https://github.com/ataulm/skipper) for clues on what’s coming up, and as always, [ping me on Twitter](https://twitter.com/ataulm) with questions and comments.

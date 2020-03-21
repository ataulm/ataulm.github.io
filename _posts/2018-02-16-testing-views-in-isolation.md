---
title: Testing Views in Isolation
# featured_image: /images/article-directory/featured-image.png
excerpt: I gave a presentation at MOBOS conference in Cluj-Napoca, Romania this week. It was based off some work I did last year, but have since added a lot of new features. The slides didn’t have any text, so here’s a summary of what I presented.

---

I gave a presentation at [MOBOS](http://romobos.com/) conference in Cluj-Napoca, Romania this week. It was based off some work I did last year, but have since added a lot of new features.

The [slides didn’t have any text](https://twitter.com/ataulm/status/964254969896595458), so here’s a **summary** of what I presented.

We’re working on an app for a video-on-demand/TV catch-up service.

![](/images/testing-views-in-isolation/tv.png)

Our team includes developers, QA and design. There’s strict process about PR review.

![](/images/testing-views-in-isolation/team.png)

We follow something similar to [gitflow](http://nvie.com/posts/a-successful-git-branching-model/) though we do not allow direct pushes to the feature branch. Instead, we commit to “working branches”, raising pull requests against the feature branch that must be code reviewed.

![Git branching diagram showing branching from develop to working branches. Merges back to develop go via a feature branch, then to the develop.](/images/testing-views-in-isolation/gitflow.png)

Once all subtasks are complete and the feature has been implemented, that’s where QA come in.

![](/images/testing-views-in-isolation/baseballbat.png)

We raise a PR to merge the feature branch into develop, and this is where QA will test the feature to ensure it meets the acceptance criteria and does not introduce regressions.

It’s a lot of effort–there are automated tests which are run as well as new ones which are added. The majority of the regressions come in the form of UI related bugs: either unintended changes in appearance, or elements which which should be displayed not displaying and vice versa.

![](/images/testing-views-in-isolation/tripod.png)

To reduce the number of UI regressions:

- Views should be as dumb as possible, only receiving the information it needs and no more as this will prevent us from binding the wrong data
- the creation/conversion of these specialised models should be done in a business layer which can be unit tested, not in the view itself

The first part is difficult for QA to check from the outside of the app, since they will need to provide many variations of mocked network responses to validate the app behaviour. For us, it’s easier since we can create these view models in our tests, and make assertions using Espresso.

Here’s what typical Espresso test looks like:

```java
@RunWith(AndroidJUnit4.class)
public class SeasonActivityTest {
    @Rule
    ActivityTestRule activityRule = new ActivityTestRule<>(SeasonActivity.class);

    @Test
    public void myTest() {
        // ...
    }
}
```

The `ActivityTestRule` is an example of a [JUnit Rule](https://stackoverflow.com/questions/13489388/how-does-junit-rule-work). It creates and launches an activity (`SeasonActivity` in this case) before each test, and finishes it after each test has completed running.

We wrote a `ViewTestRule` that extends the activity one. You pass in a layout resource, and the rule will launch an empty activity, inflate and add the layout to the root, and then otherwise behave as the `ActivityTestRule`.

```java
@RunWith(AndroidJUnit4.class)
public class EpisodeItemViewTest {
    @Rule
    ViewTestRule viewRule = new ViewTestRule<>(R.layout.item_view_episode);

    @Test
    public void myTest() 
        // ...
    }
}
```

This is not enough on its own (in most cases) because the inflated view will have no data bound to it. We can instantiate the class the binds data to the view, in this case, `EpisodeItemViewHolder`, passing the subviews as dependencies using `viewRule.getView()` to get the root.

```java
@Rule
ViewTestRule viewRule = new ViewTestRule<>(R.layout.item_view_episode);

EpisodeItemViewHolder holder;

@Before
public void setup() {
    View view = viewRule.getView();
    holder = new EpisodeItemViewHolder(
        view,
        view.findViewById(R.id.title),
        view.findViewById(R.id.subtitle),
        view.findViewById(R.id.badge),
        view.findViewById(R.id.poster)
    );
}
```

Now we can write a test! We use a `TestEpisodeBuilder` which has default values, then modify the ones that are important for that test.

```java
@Test
public void hideSubtitle_ifNotPresent() {
    Episode episode = new TestEpisodeBuilder().withoutSubtitle().build();

    viewRule.runOnMainSynchronously(view -> holder.bind(episode));
    
    onView(withId(R.id.subtitle)).check(matches(not(isDisplayed())));
}
```

The `viewRule.runOnMainSynchronously(...)` is necessary because when the holder binds, it’s modifying the view. Since the view was created on the main thread, it can only be modified on that thread.

There is an annotation that is meant to run tests completely on the main thread, which I thought would be fine (`@UiThread`) but although I don’t get any crashes with it, the test does not complete–it just runs until it’s cancelled (I didn’t wait for it to time-out). So that’s the reason we wrap the bind in that method explicitly.

Since we’re just testing that our Views are wired up correctly, we can use mocks to make sure that interactions are also bound as expected:

```java
@Test
public void clickingOnItemView_invokesListener() {
    Listener listener = Mockito.mock(Listener.class);
    Episode episode = new TestEpisodeBuilder().withListener(listener).build();

    viewRule.runOnMainSynchronously(view -> holder.bind(episode));
    onView(ViewTestRule.underTest()).perform(click());

    Mockito.verify(listener).onClickEpisodeWithId(episode.id());
}
```

We use the `click()` action from Espresso to click on the item view and then verify that the correct method was invoked with the expected parameter.

---

Our partner wanted to add better support in the app for users with disabilities.

![Person with one prosthetic runner’s blade, person in wheelchair holding basketball, person with no arms wearing swimming goggles and swimming cap](/images/testing-views-in-isolation/athletes.png)

Awesome! How can you go about such a request in your own app? I would start with what you know.

Android developers will have learned to use sp (scaleable pixels) for text instead of dp (density-independent pixels). This means that when the user increases the system font size, text that has size specified in sp will grow, whereas text with size specified in dp will remain the same.

Font size - default | Largest
---|---
![](/images/testing-views-in-isolation/default-font-size.png) | ![](/images/testing-views-in-isolation/large-text-size.png)

A common defensive mechanism for ensuring that text does not get clipped in fixed dimension layouts is setting the `android:ellipsized="end"` property on the textview:

![Side by side images of two views, one with ellipsized text to indicate text is longer than can fit in the view](/images/testing-views-in-isolation/ellipsized-text.png)

Although this makes the clipping look intentional, it means that text is not readable anymore. Instead, we should try to make our layouts responsive. The simplest way to do this is to use the `wrap_content` value for the container. This is sometimes not possible–consider whether using `sp` to define View dimensions is reasonable, as this will allow the container to grow with the text:

![Side by side images of two views, one with larger text and a proportionately larger view such that no truncation is necessary](/images/testing-views-in-isolation/growing-containers.png)

Check periodically that your app doesn’t break with variations in system font size.

We wrote a rule that can change the system font scale. Combining an `ActivityTestRule` or `ViewTestRule` with [screenshots](https://developer.android.com/reference/android/support/test/runner/screenshot/Screenshot.html), we can get a set of pictures with our app at different font scales to check if any of our layouts break:

```java
ActivityTestRule activityRule = new ActivityTestRule<SeasonActivity>(SeasonActivity.class) {
    @Override
    public void finishActivity() {
        SystemClock.sleep(160); // keep on-screen long enough to capture
        screenShotter.capture("episode-huge") // with file prefix
        super.finishActivity();
    }
};

@Rule
RuleChain ruleChain = RuleChain
    .outerRule(FontScaleRules.hugeFontScaleTestRule())
    .around(activityRule);
```

Consider how users of assistive technologies will interact with and navigate through your app. These indirect interaction methods include:

- remote controls: navigation using d-pad, back and select button
- TalkBack: the user can use touch to interact with their device, but all gestures and input is interpreted by TalkBack, and accessibility events dispatched to Views
- Switch Access: the user can assign hardware buttons to subset of navigation actions like next, previous and select. The service again dispatches (some) accessibility events to Views (e.g. click)
- VoiceAccess: the service labels actionable views and the user can indicate which view to action via voice input

![A Switch Access device, with two buttons labelled “next” and “ok”](/images/testing-views-in-isolation/switch-access.png)

For many of these indirect interaction methods, navigation through an app is linear or sequential, focusing on actionable items.

This can be frustrating when dealing with a list of views that each have inline actions.

Consider a tweet, which has six inline actions in addition to the item view action:

![sketch of a tweet](/images/testing-views-in-isolation/tweet.png)

![same tweet with seven available actions indicated by bounding boxes](/images/testing-views-in-isolation/tweet-actions.png)

Navigation using a Switch Access device through tweets in the feed would take seven steps per tweet.

We can inflate a different layout (or modify the existing layout) if we detect an accessibility service is running or if the view is in non-touch mode, removing all the inline actions, which means navigation would take one step per tweet:

![Tweet with no inline actions (retweet and like badges are displayed but these no longer function as actions, only indicators)](/images/testing-views-in-isolation/tweet-simplified.png)

Of course it’s no good _removing_ all the actions–we’re trying to make our app **accessible** and **usable** so we can’t just remove access to features. Instead, we present them in a list dialog when clicking on any item view:

![List dialog of available actions for a given tweet](/images/testing-views-in-isolation/view-dialog.png)

After doing this for each list in the app with inline actions on item views, imagine QA’s reaction to having been told they’ll have to test it and add it to their regression test procedures.

![Person with hat, backpack and glasses, with hands held up against the side of their head to indicate an inability to deal with this right now](/images/testing-views-in-isolation/explorer.png)

Well, we can test some of this now with Espresso. Although we’ve had a test rule that turns TalkBack on and off programmatically for a while, we’ve recently added the `AccessibilityRules` factory.

This contains rules for known accessibility services which we can turn on and off–this lets us test any custom behaviour in our Activities and Views.

```java
ViewTestRule viewRule = new ViewTestRule<>(R.layout.item_view_episode);

@Rule
RuleChain ruleChain = RuleChain
    .outerRule(AccessibilityRules.createTalkBackTestRule())
    .around(viewRule);
```

I told a story in which I found it difficult to put my bins out:

- it’s recycling or trash, on alternate weeks
- the bins are a similar color, both green, but it’s only easily distinguishable during the day
- there are labels, but these are difficult to read also because they are too small

The story concluded with the point that we should test our apps in various situations, e.g. in areas with lots of light and areas with very little light, since this can raise issues with color contrast.

![Two wheeled-bins side by side, with a barely distinguishable change in color from each other](/images/testing-views-in-isolation/wheeley-bins.png)

Color contrast is a measurement of the difference in lightness between two colors.

It ranges from 1:1, which is for two colors that are the same, to 1:21, which is the contrast ratio of black and white (or white and black).

Low contrast | High contrast
---|---
![white and grey, with a ratio of 1:1.2](/images/testing-views-in-isolation/contrast-high.png) | ![white and black, with a ratio of 1:21](/images/testing-views-in-isolation/contrast-low.png)

We’re working on adding a TextView assertion to grade the color contrast given the background of the test. This is difficult because:

- the color of the background may be contributed to by multiple elements because of semi-transparent colors
- there’s no guarantee that each of the elements behind the text has solid color backgrounds, vs. gradient backgrounds or images

But we can probably do some naive implementation that will work for 80% of the cases.

Then I summed up the three things you can start to look at in your app and how to test them:

- alternative interfaces when the user is interacting indirectly
- responsive layouts
- color contrast ratio

![](/images/testing-views-in-isolation/considerations.png)

The things I’m still working on/looking to understand:

- emulating user interactions like gestures that are captured and interpreted by accessibility services, like TalkBack
- adding a `ViewAssertion` that would determine whether the text from a textview has a great enough contrast with its background, based on the size of the text
- try to grant the `WRITE_SECURE_SETTINGS` permission at runtime with an adb command inside the accessibility test rules, like we do for `WRITE_SETTINGS` for the `FontScaleRule`
- understanding why the `UiThread` annotation doesn’t work as I expect (to get rid of `viewRule.runOnMainSynchronously(...)`)

Start using the [espresso-support](http://github.com/novoda/espresso-support) library today and [let me know](https://twitter.com/ataulm) what you think.

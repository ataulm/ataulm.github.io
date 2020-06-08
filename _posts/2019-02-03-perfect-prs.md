---
title: Perfecting Process for Presenting PRs
# featured_image: /images/article-directory/featured-image.png
excerpt: This post demonstrates some ways we can improve our pull request process to add important information to help the reviewer understand our changes faster.

---

I’m a visual person. When I’m reviewing code, I really appreciate when pull requests add some context around how a code change affects the end-user.

![PR description for Chunks — showing before and after comparison for a UI change](/images/perfect-prs/pr-desc.png)

It’s a lot of overhead to do this for every UI-affecting code change—_unless_ you’ve got a process that doesn’t slow you down.

Then, it’s an extra minute spent on each PR for _you_, but at the benefit of saving your reviewers’ time which is way more valuable.

## Red-green-refactor

If you’re waiting until you open your PR to take a before and after screenshot, you’ll get frustrated because you’ll have to build your head branch again.

As with Test-Driven Development, we need to start with a failing test. **Take a “before” screenshot or screen recording before you write any code.**

**After you’ve implemented your changes, and you’re building/running it to verify the fix (as we never open a PR without testing, right…?), take the “after”.**

We waste no time with unnecessary builds and it helps you focus on the change you need to make, which can mean a smaller PR.

As with TDD, the refactor step is often forgotten. We’ll uphold that tradition and forget about it here too.

## Saving screenshots and making screen recordings

The Logcat tab in Android Studio lets you take screenshots (camera icon) and record video (play icon) for the selected device:

![](/images/perfect-prs/logcat-camera.png)

With screenshots, I save them with the default settings to Desktop. If it’s the “after” screenshot, you can also “Copy to Clipboard” and paste it directly into GitHub, rather than saving it directly.

For screen recordings, I keep “Show taps” enabled on my device (you can find this under “Developer options”)—I found that the “Show taps” checkbox that’s available when you start a screen recording via Logcat messes up the device setting.

I use default settings for screen recordings too, and I make sure that I pause at the start and end of the video for a few seconds so that there’s time for people’s eyes to settle before the action begins. Since GIFs will loop, it’s also nice if you end the recording on the same screen or position as you started it.

## Converting screen recordings to GIF

[Sebastiano Poggi wrote a nifty script](https://github.com/rock3r/giffify) that uses ffmpeg to convert videos to GIFs — unfortunately, GitHub doesn’t allow uploading video files even though they can sometimes be smaller.

Clone the repository somewhere safe, and add it to your path:

```bash
export GITHUB=~/src/github.com
export PATH=$PATH:$GITHUB/rock3r/giffify
```

Then you can use the `giffify.py` script anywhere to convert a video file:

```bash
giffify.py ~/Desktop/before.mp4 -dh 640
```

and it’ll save `before.gif` in the same place as the source file.

Setting a desired height of 640 pixels means it’s usually small enough for GitHub’s size limits, and also small enough not to overshadow other parts of the description.

## Prettier screenshots

If you need it to look fancy, e.g. for a slide deck for clients or a presentation, you can use this script (thanks [Lara Martín](https://medium.com/u/42754ec47023) and [Emma Guy](https://medium.com/u/64263b17ad9d)!) to clean up the status bar.

```bash
#!/bin/sh

# source: https://android.jlelse.eu/clean-your-status-bar-like-a-pro-76c89a1e2c2f
# Add these aliases to your `~/.profile`
# alias demoOn='sh /Users/<username>/scripts/adb-demo.sh on'
# alias demoOff='sh /Users/<username>/scripts/adb-demo.sh off'

CMD=$1

if [[ $CMD != "on" && $CMD != "off" ]]; then
  echo "Usage: $0 [on|off]" >&2
  exit
fi

adb shell settings put global sysui_demo_allowed 1 

if [ $CMD == "on" ]; then
  // display time 11:10
  adb shell am broadcast -a com.android.systemui.demo -e command clock -e hhmm 1110

  // Display full mobile data without type
  adb shell am broadcast -a com.android.systemui.demo -e command network -e mobile show -e level 4 -e datatype false

  // Hide notifications
  adb shell am broadcast -a com.android.systemui.demo -e command notifications -e visible false

  // Show full battery but not in charging state
  adb shell am broadcast -a com.android.systemui.demo -e command battery -e plugged false -e level 100
elif [ $CMD == "off" ]; then
  adb shell am broadcast -a com.android.systemui.demo -e command exit
fi
```

Using the aliases you can enable and disable demo mode on your device with `demoOn` or `demoOff`.

## Markdown tables

Side-by-side comparisons are really useful for screenshots or very short GIFs.

```
Before | After
---|---
<image1> | <image2>
```

The syntax is a bit finicky, but when you get used to it, you can type it _while_ the images are uploading, then cut and paste the URLs into the correct places.

```
Before | After
---|---
![before](https://user-images.githubusercontent.com/2678555/50713461-fc7be080-106c-11e9-8238-629dba73b4cd.png) | ![after](https://user-images.githubusercontent.com/2678555/50713457-fc7be080-106c-11e9-9766-036dc58e71ac.png)
```

Use the same data-set if possible to make the image-diff as clear as possible.

---

Please delete the screenshots from your desktop afterwards:

<center>
<blockquote class="twitter-tweet"><p lang="und" dir="ltr">🤮🤮🤮🤮🤮 <a href="https://twitter.com/emmaguy?ref_src=twsrc%5Etfw">@emmaguy</a> <a href="https://t.co/96ylIhncHw">pic.twitter.com/96ylIhncHw</a></p>&mdash; ataúl ✏️ (@ataulm) <a href="https://twitter.com/ataulm/status/1067056948263170048?ref_src=twsrc%5Etfw">November 26, 2018</a></blockquote>
</center>

Do you have other tips for a speedy PR description or tools to help you in this? Let me know [on Twitter](https://twitter.com/ataulm)!

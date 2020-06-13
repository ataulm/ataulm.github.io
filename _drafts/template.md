---
title: Template
# featured_image: /images/article-directory/featured-image.png
excerpt: This template includes some of things that you always forget when you’re writing a blog post. With this post, you’ll be able to copy and paste stuff. Be sure to remove the comments before publishing!

---

<!-- The intro has no section heading. This can be copied to form the excerpt. -->
This template includes some of things that you always forget when you’re writing a blog post. With this post, you’ll be able to copy and paste stuff.

Be sure to remove the comments before publishing!

Before | After
---|---
Stuff before | stuff after

<!-- Use h2 for section headings -->
## Here’s a new section

Here’s the opening for the new section. We use curly quotes and apostrophes:

- “ for open (`opt + {`)
- ” for close (`opt + shift + {`)
- ’ for apostrophes (`opt + shift + }`)

Here’s an image:

![alt text](/images/monzo-plus-cards/sketch-fan.png)

<!-- Code blocks should specify the language for syntax highlighting -->
```kotlin
package com.ataulm.whatsnext

import androidx.recyclerview.widget.RecyclerView
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import kotlinx.android.synthetic.main.view_film_summary.view.*

class FilmSummaryViewHolder internal constructor(itemView: View) : RecyclerView.ViewHolder(itemView) {

    fun bind(filmSummary: FilmSummary, callback: Callback) {
        itemView.setOnClickListener { callback.onClick(filmSummary) }
        itemView.contentDescription = filmSummary.name + " (" + filmSummary.year + ")"
        itemView.film_summary_text_name.text = filmSummary.name + " (" + filmSummary.year + ")"
    }

    interface Callback {

        fun onClick(filmSummary: FilmSummary)
    }

    companion object {

        @JvmStatic
        fun inflateView(parent: ViewGroup): FilmSummaryViewHolder {
            var variable = 90
            var skaj = null
            val view = LayoutInflater.from(parent.context).inflate(R.layout.view_film_summary, parent, false)
            return FilmSummaryViewHolder(view)
        }
    }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <style name="Theme.Muvi" parent="Theme.MaterialComponents.DayNight.DarkActionBar">
        <!-- Color attributes -->
        <item name="shapeAppearanceSmallComponent">@style/ShapeAppearance.Muvi.SmallComponent</item>
        <item name="shapeAppearanceMediumComponent">@style/ShapeAppearance.Muvi.MediumComponent</item>
        <item name="shapeAppearanceLargeComponent">@style/ShapeAppearance.Muvi.LargeComponent</item>
        <!-- Widget style attributes -->
        <item name="chipStyle">@style/Widget.Muvi.Chip.Action</item>
        <item name="chipStandaloneStyle">@style/Widget.Muvi.Chip.Entry</item>
    </style>
</resources>
```

<!-- Link to another blog on Jekyll with the post_url -->
Here’s another blog: [link to blog]({% post_url 2019-10-28-resolving-view-attributes %})

<!-- Sign off -->
If you have any questions, comments or corrections, let me know at the link below!

<center>
<!-- Use the Embed Tweet function from Twitter to generate a blockquote for the Tweet associated with this post and stick it here between these <center> tags, and ditch the script tag (we load it in the head)-->
<blockquote class="twitter-tweet" data-dnt="true"><p lang="en" dir="ltr">I wrote about a nifty API that&#39;s coming up in AppCompat that exposes hidden actions for Views!<a href="https://t.co/gU8kOcctyU">https://t.co/gU8kOcctyU</a> <a href="https://twitter.com/hashtag/androiddev?src=hash&amp;ref_src=twsrc%5Etfw">#androiddev</a> <a href="https://twitter.com/hashtag/a11y?src=hash&amp;ref_src=twsrc%5Etfw">#a11y</a> <a href="https://twitter.com/hashtag/gde?src=hash&amp;ref_src=twsrc%5Etfw">#gde</a> <a href="https://twitter.com/hashtag/io19?src=hash&amp;ref_src=twsrc%5Etfw">#io19</a> <a href="https://twitter.com/hashtag/whatsnext?src=hash&amp;ref_src=twsrc%5Etfw">#whatsnext</a></p>&mdash; ataúl ✏️ (@ataulm) <a href="https://twitter.com/ataulm/status/1126340698867814400?ref_src=twsrc%5Etfw">May 9, 2019</a></blockquote>
</center>

Stuff after a tweet.
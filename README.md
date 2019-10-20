Mostly followed instructions from the [official site](https://jekyllrb.com/).

- After installing Ruby with `brew`, I had to update my PATH so that the `brew` Ruby was listed before the system Ruby.
- Updated the Gemfile manually with an explicit github-pages version (`Gemfile` has more info)

Building site locally, and running a server so we can preview at [localhost:4000](http://localhost:4000/). Changes to the site will require re-running the `serve` command:

```bash
bundle exec jekyll serve
```

Building the site so that it can be pushed is done using `build`:

```bash
bundle exec jekyll build
```

Then commit and push to master.

## Writing posts

Create new posts in `_posts`. Posts should be named in the format `YEAR-MONTH-DAY-title.markdown`.

There's a metadata block at the top of each post I think is used to define (and override) attributes that the site/theme can use ([example from Jorge's blog](https://raw.githubusercontent.com/JorgeCastilloPrz/jorgecastilloprz.github.io/source/_posts/2019-10-12-dependency-inversion-on-android-theming.md)):

```
---
layout: post
current: post
cover: assets/images/painting2.jpg
navigation: True
title: Dependency Inversion on Android Theming
date: 2019-10-11 12:36:00
tags: [android, kotlin]
class: post-template
subclass: 'post'
author: jorge
---
```

## Theming

I bought a theme called [Simples](https://jekyllthemes.io/theme/simples-blog-jekyll-theme) which I've saved in a [private repository here](https://github.com/ataulm/ataulm.github.io-simples) (due to the license, can't keep the source of the theme in this public repo).


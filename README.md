Mostly followed instructions from the [official site](https://jekyllrb.com/).

- After installing Ruby with `brew`, I had to update my PATH so that the `brew` Ruby was listed before the system Ruby.
- Updated the Gemfile manually with an explicit github-pages version (`Gemfile` has more info)

Building site locally, and running a server so we can preview at [localhost:4000](http://localhost:4000/). Changes to the `_config` will require re-running the `serve` command but otherwise it'll auto-reload:

```bash
bundle exec jekyll serve
```

Building the site so that it can be pushed is done using `build`:

```bash
bundle exec jekyll build
```

Then commit and push to master. GitHub will automatically generate the `_site` so there's no need to commit this (see `.gitignore`). Though the first build will take about 10 minutes, subsequent pushes seem to update within a minute, then the updated content should be available on [ataulm.github.io](https://ataulm.github.io).

## Writing posts

Create new posts in `_posts`. Posts should be named in the format `YEAR-MONTH-DAY-title.markdown`.

We can add a [front matter block](https://jekyllrb.com/docs/front-matter/) at the top of each post (or any file) where we can override custom attributes or define new ones:

```
---
layout: post
title: Blogging Like a Hacker
---
```

The `date` (UTC) in a front matter block can be used to override the date from the file name, and posts dated in the future (at the time the site is generated) won't be shown.
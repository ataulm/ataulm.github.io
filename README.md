Mostly followed instructions from the [official site](https://jekyllrb.com/).

- After installing Ruby with `brew`, I had to update my PATH so that the `brew` Ruby was listed before the system Ruby.
- Updated the Gemfile manually with an explicit github-pages version (`Gemfile` has more info)

Building site locally, and running a server so we can preview at [localhost:4000](http://localhost:4000/)

```bash
bundle exec jekyll serve
```

Changes to the site will require re-running the `serve` command.

Building the site so that it can be pushed is done using `build`:

```bash
bundle exec jekyll build
```

Then commit and push to master.
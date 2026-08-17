# gdubya.github.io

Source for [garethwestern.com](https://garethwestern.com) — a Jekyll site built
by GitHub Pages using the [so-simple-theme][so-simple] remote theme.

## Notes to self

### Local preview

```sh
bundle install
bundle exec jekyll serve
```

Two things that will bite you on a fresh Debian/Ubuntu box:

- `ruby-full` installs the binaries version-suffixed, so it's `bundle3.2`, not
  `bundle`.
- Don't `bundle config set path vendor/bundle`. Jekyll's default `exclude` for
  `vendor/bundle/` doesn't stop it recursing into the installed gems, and the
  build dies on Jekyll's own `site_template` fixtures. Install to a path
  *outside* the repo instead.

### Upgrade dependencies

```sh
bundle update

gem install --user-install bundler-audit   # not in the Gemfile
bundle-audit check --update
```

Dependabot (`.github/dependabot.yml`) opens a weekly PR for gem and Action
updates, so this is mostly only needed when you want to bump things by hand.

Note that `Gemfile.lock` only affects *local* builds. The live site is built by
GitHub Pages with its own pinned gem set — see
<https://pages.github.com/versions/>.

### CI

`.github/workflows/ci.yml` runs on every push and PR, plus weekly:

- builds the site, so a bad `_config.yml` or Liquid error fails fast
- checks internal links with html-proofer (informational only)
- runs `bundle audit` against the Ruby advisory DB

[so-simple]: https://github.com/mmistakes/so-simple-theme

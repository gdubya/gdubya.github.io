# gdubya.github.io

Source for [garethwestern.com](https://garethwestern.com) — a Jekyll site built
with the [Chirpy][chirpy] theme, deployed via a GitHub Actions workflow.

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

Unlike the old `so-simple-theme` setup, the live site is now built and
deployed entirely from this repo's own `Gemfile.lock` via GitHub Actions —
it's no longer subject to GitHub Pages' legacy allowed-gems list
(<https://pages.github.com/versions/>). GitHub Pages is configured to deploy
from "GitHub Actions" (Settings → Pages), not from a branch.

### CI / deploy

- `.github/workflows/ci.yml` runs on every PR, plus weekly: builds the site
  (so a bad `_config.yml` or Liquid error fails fast), checks internal links
  with html-proofer, and runs `bundle audit` against the Ruby advisory DB.
- `.github/workflows/pages-deploy.yml` runs on every push to `main`: builds
  the site, checks internal links, and deploys the result to GitHub Pages.

### Content structure

- Posts live in `_posts/`, one category (`categories: [x]`) plus any number of
  `tags: [...]` per post; permalinks are `/posts/:title/`.
- Static pages (About, Archives, Categories, Tags) live in `_tabs/`.
- Old date-based post URLs (from before the Chirpy migration) redirect to
  their new `/posts/:title/` URL via `redirect_from:` front matter
  (`jekyll-redirect-from`).

[chirpy]: https://github.com/cotes2020/jekyll-theme-chirpy


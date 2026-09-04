# kevbuchanan.github.io

Jekyll source for the site at <https://kevbuchanan.github.io>. Pushing to
`main` triggers `.github/workflows/pages.yml`, which builds with the
Gemfile's Jekyll and deploys to GitHub Pages.

## Local development

```
bundle install
bundle exec jekyll serve
```

Include unpublished drafts from `_drafts/`:

```
bundle exec jekyll serve --drafts
```

## Writing

Posts live in `_posts/` as `YYYY-MM-DD-slug.md` and need only a `title` in
their front matter. Everything else comes from the defaults in `_config.yml`.

## Styles

`assets/core.scss` pulls in four partials from `_sass/`:

- `tokens` -- font faces plus the custom properties for type, spacing, and
  both color themes
- `base` -- element defaults
- `layout` -- header, post list, and article structure
- `syntax` -- Rouge highlighting, driven by the palette in `tokens`

Colors follow `prefers-color-scheme`. Public Sans is self-hosted from
`assets/fonts/` under the SIL Open Font License, included as
`assets/fonts/OFL.txt`.

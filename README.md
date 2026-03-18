# programming-practices

The `docs/` directory is configured for GitHub Pages via [`.github/workflows/pages.yml`](./.github/workflows/pages.yml).

Add a new published page by creating a Markdown file in [`docs/`](./docs/) with:

```md
---
title: Your Page Title
description: One-sentence summary for the docs index.
layout: default
---
```

It will be rendered by Jekyll and linked automatically from [`docs/index.md`](./docs/index.md).

## Local Preview

Bundler is configured to install gems into `vendor/bundle` for this repo, so it does not need to write to the system gem path.

Install the local Jekyll dependencies:

```bash
bundle install
```

Then serve the docs site locally:

```bash
bundle exec jekyll serve --source docs
```

The local site will usually be available at `http://127.0.0.1:4000/`.

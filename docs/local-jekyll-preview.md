---
title: Local Jekyll Preview
description: How to install gems locally with Bundler and run the docs site on your machine.
layout: default
pin_bottom: true
hide_from_index: true
---

# Local Jekyll Preview

This repo is configured so Bundler installs gems into `vendor/bundle` instead of the system gem directory.

That avoids permission errors like writes to `/var/lib/gems/...`.

## Install dependencies

From the repo root:

```bash
bundle install
```

## Run the docs site locally

```bash
bundle exec jekyll serve --source docs
```

The site will usually be available at `http://127.0.0.1:4000/`.

## Why Bundler was trying to use a global location

Without a repo-local Bundler path, your system Ruby and Bundler may default to the system gem home, often under `/var/lib/gems/...`.

That directory is usually owned by root, so `bundle install` fails with a permission error.

This repo now includes [`.bundle/config`](../.bundle/config) with:

```yaml
---
BUNDLE_PATH: "vendor/bundle"
```

## Check the active Bundler config

```bash
bundle config
```

You should see the local `path` set to `vendor/bundle`.

## If Bundler still uses the system path

Run this once from the repo root:

```bash
bundle config set --local path vendor/bundle
```

Then run:

```bash
bundle install
```

## Useful cleanup

If you want to reinstall the gems cleanly:

```bash
rm -rf vendor/bundle
bundle install
```

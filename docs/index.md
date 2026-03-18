---
title: Docs
---

# Docs

This index is generated automatically from the published pages in this directory.

{% assign doc_pages = site.html_pages | where_exp: "item", "item.url != '/'" | sort: "title" %}

{% if doc_pages.size > 0 %}
<div class="doc-list">
{% for page in doc_pages %}
  <article class="doc-card">
    <h2><a href="{{ page.url | relative_url }}">{{ page.title | default: page.name | replace: '.html', '' }}</a></h2>
    {% if page.description %}
      <p>{{ page.description }}</p>
    {% endif %}
    <p class="doc-meta">
      <strong>{% if page.layout == "slides" %}Slides{% else %}Article{% endif %}</strong>
      <span>·</span>
      <code>{{ page.path | default: page.name }}</code>
    </p>
  </article>
{% endfor %}
</div>
{% else %}
No published docs found yet.
{% endif %}

## Adding a New Page

1. Create a new Markdown file in `docs/`.
2. Start it with this front matter:

```md
---
title: Your Page Title
description: One-sentence summary for the docs index.
layout: default
---
```

Use `layout: slides` for a slide deck.

3. Separate slides with `---` when using the slides layout.
4. Commit and push. The page will be published and added here automatically.

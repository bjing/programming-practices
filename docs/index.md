---
title: Docs
---

# Docs

This index is generated automatically from the Markdown pages in this directory.

{% assign doc_pages = site.pages | where_exp: "item", "item.name != 'index.md'" | where_exp: "item", "item.ext == '.md'" | sort: "title" %}

{% if doc_pages.size > 0 %}
{% for page in doc_pages %}
- [{{ page.title | default: page.name | replace: '.md', '' }}]({{ page.url | relative_url }})
{% endfor %}
{% else %}
No published docs found yet.
{% endif %}

## Adding a New Page

1. Create a new Markdown file in `docs/`.
2. Start it with this minimal front matter:

```md
---
title: Your Page Title
---
```

3. Commit and push. The page will be published and added here automatically.

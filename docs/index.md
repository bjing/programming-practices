---
title: Programming Practices
---

This index is generated automatically from the published pages in this directory.

{% assign doc_pages = site.html_pages | where_exp: "item", "item.url != '/'" | where_exp: "item", "item.hide_from_index != true" | sort: "title" %}
{% assign main_pages = doc_pages | where_exp: "item", "item.pin_bottom != true" %}
{% assign utility_pages = doc_pages | where_exp: "item", "item.pin_bottom == true" %}

{% if doc_pages.size > 0 %}
<div class="doc-list">
{% for page in main_pages %}
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
{% for page in utility_pages %}
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

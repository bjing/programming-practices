---
title: How to add a new page
description: Instructions on how to add a new document page/slides
layout: default
---

## Adding a New Page

0. Create a new Markdown file in `docs/`.
1. Start it with this front matter:

```md
---
title: Your Page Title
description: One-sentence summary for the docs index.
layout: default
---
```

Use `layout: slides` for a slide deck.

2. Separate slides with `---` when using the slides layout.
3. Commit and push. The page will be published and added here automatically.

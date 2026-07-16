---
layout: post
title: "Hello World"
date: 2026-07-16 10:30:00 +0530
description: "A short summary shown on the blog list and in link previews."
image: /assets/images/2026-07-16-hello-world/hero.png
tags: [meta, setup]
---

This is a post. Everything below the `---` block is plain markdown.

## Front matter, explained

| Field | Required? | What it does |
|---|---|---|
| `layout` | yes | Always `post`. |
| `title` | yes | Shown on the list and the post page. |
| `date` | yes | Controls ordering **and** the displayed time. Keep `+0530`. |
| `description` | no | Blurb on the blog list + link previews. Falls back to the first lines of the post. |
| `image` | no | Thumbnail on the list + link preview image. |
| `tags` | no | Little pills under the title. |

## Adding images

Point at a file you've uploaded to `assets/images/`:

```markdown
![Alt text](/assets/images/2026-07-16-hello-world/hero.png)
```

Which renders as:

![A placeholder hero image](/assets/images/2026-07-16-hello-world/hero.png)

Need to control the width? Use plain HTML — Jekyll passes it straight through:

```html
<img src="/assets/images/2026-07-16-hello-world/hero.png" alt="Diagram" width="500">
```

Want a caption?

```html
<figure>
  <img src="/assets/images/my-post/diagram.png" alt="Diagram">
  <figcaption>Figure 1 — the thing.</figcaption>
</figure>
```

## The rest of markdown

**Bold**, *italic*, `inline code`, and [links](https://example.com).

> Blockquotes look like this.

```python
def hello():
    return "code blocks get syntax highlighting"
```

1. Numbered lists
2. Work fine
   - So do nested ones

---

Delete this file once you've got the hang of it.

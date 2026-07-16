# agenticworld.co.in

Jekyll blog on GitHub Pages. No build step, no CI, no Node.
Publish a post by adding one markdown file.

---

## Part 1 — One-time setup

### 1. Create the repo

On github.com → **New repository**

- **Name:** `<your-username>.github.io` — exactly that, using your real username.
- **Visibility:** Public (required for free GitHub Pages).
- Don't add a README/`.gitignore`/license — this folder already has them.

### 2. Upload this folder

**Web (no git needed):**
Open the empty repo → **uploading an existing file** → drag in the *contents* of
this folder (not the folder itself) → Commit.

> ⚠️ GitHub's web uploader skips dotfiles. After uploading, use
> **Add file → Create new file** to hand-create `.gitignore` and paste its
> contents in. Or just skip it — it's cosmetic.

**Command line:**
```bash
cd agenticworld
git init -b main
git add -A
git commit -m "Initial site"
git remote add origin https://github.com/<your-username>/<your-username>.github.io.git
git push -u origin main
```

### 3. Edit `_config.yml`

At minimum change `author.email`. Uncomment the social handles you want.

### 4. Point GoDaddy at GitHub

GoDaddy → **My Products → DNS → Manage Zones** → `agenticworld.co.in`

**Delete** the parked `A` record on `@` and any parking `CNAME`. Then add:

| Type  | Name | Value                        | TTL     |
|-------|------|------------------------------|---------|
| A     | @    | `185.199.108.153`            | 1 hour  |
| A     | @    | `185.199.109.153`            | 1 hour  |
| A     | @    | `185.199.110.153`            | 1 hour  |
| A     | @    | `185.199.111.153`            | 1 hour  |
| CNAME | www  | `<your-username>.github.io.` | 1 hour  |

### 5. Turn on Pages

Repo → **Settings → Pages**

- Source: **Deploy from a branch** → `main` → `/ (root)`
- Custom domain: `agenticworld.co.in` → Save
- Wait for the DNS check to go green (minutes, occasionally hours)
- **Then** tick **Enforce HTTPS**

The `CNAME` file in this folder means the custom domain field should
already be pre-filled. Don't delete that file.

---

## Part 2 — Publishing a post

1. Repo → `_posts` folder → **Add file → Create new file**
2. Name it `2026-07-20-my-post-title.md` — the `YYYY-MM-DD-` prefix is
   **mandatory**, Jekyll ignores files without it.
3. Paste from `_drafts/TEMPLATE.md`, fill it in.
4. **Commit changes.** Live in ~60 seconds.

Works from your phone's browser.

### Front matter reference

```yaml
---
layout: post                                    # always "post"
title: "Your Title"                             # required
date: 2026-07-20 09:00:00 +0530                 # required — controls order + display
description: "Blurb for the list and previews." # optional
image: /assets/images/my-post/hero.png          # optional thumbnail
tags: [architecture, agents]                    # optional
---
```

Newest post appears at the top automatically — that's driven by `date:`,
not the filename. Include the time so same-day posts order correctly.

---

## Part 3 — Images

### The fast way
Drag an image into the GitHub web editor while writing. It uploads to
GitHub's CDN and inserts the markdown for you. Zero setup — but the file
lives outside your repo, so those links break if you ever migrate.

### The durable way
1. `assets/images` → **Add file → Upload files**
2. Use a subfolder per post: `assets/images/2026-07-20-my-post/`
3. Reference it:
   ```markdown
   ![Alt text](/assets/images/2026-07-20-my-post/hero.png)
   ```

**Compress first.** A 4 MB phone photo will make the page crawl.
[Squoosh](https://squoosh.app) or [TinyPNG](https://tinypng.com) — aim under 300 KB.
Hero images look best at 1200×630.

Two placeholder images ship with this folder — `assets/images/profile.jpg`
and the hello-world hero. Replace both.

---

## Part 4 — Adding a third tab

1. Create `notes.md` at the repo root:
   ```markdown
   ---
   layout: page
   title: Notes
   permalink: /notes/
   ---

   Content here.
   ```
2. Add it to `header_pages` in `_config.yml`:
   ```yaml
   header_pages:
     - index.md
     - about.md
     - notes.md
   ```

Nav order follows the list order.

---

## Part 5 — Local preview (optional)

Only if you want to see changes before pushing. Requires Ruby 3.x.

```bash
gem install bundler
bundle install
bundle exec jekyll serve --livereload --drafts
```

→ http://localhost:4000

Skip this entirely if you're happy editing on github.com.

---

## What each file does

```
_config.yml                  Site settings + the tab list (header_pages)
index.md                     Tab 1 — blog listing
about.md                     Tab 2 — about you
CNAME                        Tells GitHub your custom domain. Don't delete.
_layouts/home.html           Overrides Minima's list layout (thumbnails, time)
_posts/                      Your posts live here. One .md per post.
_drafts/TEMPLATE.md          Copy this when writing a new post. Not published.
assets/css/style.scss        Your CSS on top of Minima's
assets/images/               Your images
Gemfile                      Local preview only. GitHub Pages ignores it.
```

---

## Gotchas

- **Post not showing?** Check the filename prefix (`YYYY-MM-DD-`) and that
  `date:` isn't in the future — future-dated posts are hidden until then.
- **CSS not applying?** `assets/css/style.scss` must keep its empty `---`
  front matter block at the top. Without it, Jekyll copies the file raw
  instead of compiling it.
- **Build failed?** Repo → **Actions** tab shows the error. Usually a YAML
  typo in front matter — an unescaped `:` in a title is the classic one.
  Wrap titles in quotes.
- **Plugins:** only GitHub's
  [allowlist](https://pages.github.com/versions/) works with this setup.
  Anything outside it means switching to a GitHub Actions build — same repo,
  same posts, just an extra workflow file. Nothing here would be wasted.

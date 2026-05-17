# Blog Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Switch `lee411806.github.io` from minima to Minimal Mistakes (air skin) with a left sidebar showing author profile and category navigation.

**Architecture:** Replace `_config.yml` with Minimal Mistakes remote_theme config, add `_data/navigation.yml` for nav/sidebar, add `_pages/` for About and Categories, replace `index.md` with `index.html`.

**Tech Stack:** Jekyll, GitHub Pages, mmistakes/minimal-mistakes remote theme

---

## Files to Create or Modify

| Action | Path | Purpose |
|--------|------|---------|
| Modify | `_config.yml` | Theme, skin, author, plugins, layout defaults |
| Create | `_data/navigation.yml` | Top nav + sidebar category links |
| Create | `index.html` | Home page (layout: home) |
| Delete | `index.md` | Replaced by index.html |
| Delete | `categories.md` | Replaced by _pages/categories.md |
| Create | `_pages/about.md` | About page |
| Create | `_pages/categories.md` | Category archive page |
| Modify | `_posts/2026-01-01-first-post.md` | Add author_profile: true |

---

### Task 1: Update _config.yml

**Files:**
- Modify: `_config.yml`

- [ ] **Step 1: Replace _config.yml with Minimal Mistakes config**

```yaml
remote_theme: mmistakes/minimal-mistakes
minimal_mistakes_skin: air

title: Jaeyong Blog
description: Backend Developer | GIS | LLM/RAG
url: "https://lee411806.github.io"

author:
  name: "Jaeyong Lee"
  bio: "Backend Developer | GIS | LLM/RAG"
  links:
    - label: "GitHub"
      icon: "fab fa-fw fa-github"
      url: "https://github.com/lee411806"

include:
  - _pages

plugins:
  - jekyll-feed
  - jekyll-include-cache

defaults:
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      sidebar:
        nav: "categories"
  - scope:
      path: "_pages"
      type: pages
    values:
      layout: single
      author_profile: true
      sidebar:
        nav: "categories"
```

- [ ] **Step 2: Verify YAML is valid**

Open `_config.yml` and confirm no indentation errors. Every `values:` block must be indented under its `scope:`.

- [ ] **Step 3: Commit**

```bash
git add _config.yml
git commit -m "feat: switch to Minimal Mistakes remote theme (air skin)"
```

---

### Task 2: Create _data/navigation.yml

**Files:**
- Create: `_data/navigation.yml`

- [ ] **Step 1: Create `_data/` directory and `navigation.yml`**

```yaml
main:
  - title: "Home"
    url: /
  - title: "About"
    url: /about/
  - title: "Categories"
    url: /categories/

categories:
  - title: "Categories"
    children:
      - title: "Backend"
        url: /categories/#backend
      - title: "GIS"
        url: /categories/#gis
```

- [ ] **Step 2: Commit**

```bash
git add _data/navigation.yml
git commit -m "feat: add main nav and sidebar category navigation"
```

---

### Task 3: Replace index.md with index.html

**Files:**
- Create: `index.html`
- Delete: `index.md`

- [ ] **Step 1: Create `index.html`**

```html
---
layout: home
author_profile: true
sidebar:
  nav: "categories"
---
```

- [ ] **Step 2: Delete `index.md`**

```bash
git rm index.md
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: replace index.md with index.html for Minimal Mistakes home layout"
```

---

### Task 4: Remove root-level categories.md

**Files:**
- Delete: `categories.md` (root level — replaced by _pages/categories.md)

- [ ] **Step 1: Remove root categories.md**

```bash
git rm categories.md
```

- [ ] **Step 2: Commit**

```bash
git commit -m "chore: remove root categories.md (replaced by _pages/categories.md)"
```

---

### Task 5: Create _pages/about.md

**Files:**
- Create: `_pages/about.md`

- [ ] **Step 1: Create `_pages/about.md`**

```markdown
---
title: "About"
permalink: /about/
---

안녕하세요, 백엔드 개발자 이재용입니다.

- Backend 개발 (Java, Python)
- GIS 공간정보 시스템
- LLM / RAG 기반 AI 서비스

GitHub: [lee411806](https://github.com/lee411806)
```

- [ ] **Step 2: Commit**

```bash
git add _pages/about.md
git commit -m "feat: add About page"
```

---

### Task 6: Create _pages/categories.md

**Files:**
- Create: `_pages/categories.md`

- [ ] **Step 1: Create `_pages/categories.md`**

```markdown
---
title: "Categories"
layout: categories
permalink: /categories/
---
```

- [ ] **Step 2: Commit**

```bash
git add _pages/categories.md
git commit -m "feat: add Categories archive page"
```

---

### Task 7: Update post front matter

**Files:**
- Modify: `_posts/2026-01-01-first-post.md`

- [ ] **Step 1: Update front matter**

```markdown
---
title: "첫 블로그 글"
categories:
  - backend
  - gis
---

안녕하세요 👋  
GitHub 블로그 시작합니다.

앞으로 개발 기록 남길 예정입니다 🚀
```

- [ ] **Step 2: Commit**

```bash
git add _posts/2026-01-01-first-post.md
git commit -m "chore: update post front matter for Minimal Mistakes"
```

---

### Task 8: Push to GitHub

- [ ] **Step 1: Push all commits**

```bash
git push origin master
```

- [ ] **Step 2: Verify**

Open `https://lee411806.github.io` in browser after ~60 seconds (GitHub Pages rebuild time). Confirm:
- Air skin (white bg, blue accent) is active
- Left sidebar shows author name, bio, GitHub link, and category nav
- Top nav shows Home / About / Categories
- Home page lists the first post

---
layout: post
title: "Blog of a blog: How I Deployed Jekyll on GitHub Pages"
date: 2025-09-04 19:00:00 -0400
categories: [Dev Notes]
tags: [jekyll, github-pages, blogging, static-sites]
description: "My plug-and-play recipe for spinning up a Jekyll blog on GitHub Pages, plus the gotchas I hit while writing my Jetson series."
image: /assets/img/posts/blog-of-a-blog/blog-cover.png
image_alt: "OSRS-style red potion labeled 'jekyll' plus the GitHub Octocat inside a pixel stone frame"
image_position: "center" # tweak if you want to reframe the crop, e.g. "50% 42%" or "top"
toc: true
toc_label: "On this page"
last_modified_at: 2025-09-04
excerpt_separator: <!--more-->
---

> **TL;DR** — I stood up a fully version-controlled blog with **Jekyll + GitHub Pages** so I own the content, the workflow, and the styling. Below is the exact path, commands, and fixes that got me online.

<!--more-->

## Publishing content

After completing the Jetson post, I was eager to share my thoughts and assumed I was approaching a quick deployment. I had reviewed my content extensively and was ready to share my fun hacktivity with the world. Unfortunately, I realized that I had not thought through a proper deployment and management strategy; Where should I host? Will it be cost-effective? Will it be maintainable/flexible?

I went down the path of exploring common blog webapps like Medium, Hashnode, and Dev.To. What I found was a combination of clunky editors, fuzzy terms, and strong opinions from various redditors. I decided to do some additional research and stumbled upon the Jekyll + GitHub pages framework

Now, I could go into a long-winded explanation on what Jekyll + GitHub pages, but I'll just quickly jot down the definition from the docs and you can dig deeper if you'd like.
'Jekyll is a static site generator with built-in support for GitHub Pages and a simplified build process.'

## Why not Hashnode (or other platforms)?

Having a better understanding of this implementation left me with a couple of conlusions.

I wanted **control** and **portability**:

- **Git workflow:** Provides a repo-first workflow where content lives within the code and every change is version controlled.
- **No lock-in:** I own the content and styling, there’s zero cost incurred, and deploys are as simple as a push.
- **Custom theming:** Allows flexibility to indroduce custom layouts and features.

---

## The 10-minute recipe

> Works with `username.github.io` repos or any repo + Pages.

1. **Create the repo**

```bash
# If you want root-domain hosting:
#   <your-username>.github.io
# Otherwise any repo name works with Pages.
```

2. **Install Jekyll locally**

```bash
gem install bundler jekyll
jekyll new my-blog
cd my-blog
bundle exec jekyll serve   # http://127.0.0.1:4000
```

3. **Pick a theme (Minima to start)**

```bash
# If you used `jekyll new`, Minima is pre-wired. Otherwise:
# Gemfile → gem "minima", "~> 2.5"
bundle install
```

4. **Connect to GitHub Pages**

```bash
git init
git add -A
git commit -m "Initial blog"
git branch -M main
git remote add origin git@github.com:<your-username>/<your-username>.github.io.git
git push -u origin main
```

5. **Turn on Pages**

- GitHub → **Settings** → **Pages** → Source: _Deploy from a branch_, Branch: `main` (or `gh-pages`), Folder: `/ (root)` (or `/docs`).

---

## Project structure I’m using

.
├── \_config.yml
├── \_posts/
│ └── 2025-09-04-blog-of-a-blog.md
├── \_layouts/
│ └── post.html
├── assets/
│ └── img/
│ └── posts/
│ └── blog-of-a-blog/
│ └── blog-cover.png
└── index.md

> **Heads-up:** `_posts` filenames must be `YYYY-MM-DD-title.md`. If something doesn’t render, check date + front matter.

---

## Hero image & sizing

I render the hero from front matter (this file’s `image:` key). In my layout I wrap it like:

```liquid
{% if page.image %}
  <figure class="post-cover" style="--pos: {{ page.image_position | default: 'center' }};">
    <img src="{{ page.image | relative_url }}"
         alt="{{ page.image_alt | default: page.title | escape }}"
         loading="eager" fetchpriority="high" decoding="async">
  </figure>
{% endif %}
```

And the CSS forces a **short, banner-style** crop:

```scss
.post-cover {
  aspect-ratio: 21 / 9;
  max-height: 360px;
  overflow: hidden;
  border-radius: 12px;
}
.post-cover img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: var(--pos, center);
}
```

Export the cover around **2048×768** for crisp results.

---

## Minimal `_config.yml`

```yml
title: "smoran-dev — Blog & Resume"
description: "Jetson, data engineering, and hands-on AI."
url: "https://<your-username>.github.io"
# baseurl: "" # set if publishing to a subpath (e.g., /blog)

theme: minima
markdown: kramdown

minima:
  skin: dark
  social_links:
    - platform: github
      user_url: https://github.com/<your-username>

# Excerpts
excerpt_separator: "<!--more-->"
```

---

## Gotchas I hit (and fixes)

- **Images not rendering:** put assets under `/assets/...` and reference with site-root paths (`/assets/...`). If using `baseurl`, prefix with `{{ site.baseurl }}`.
- **Post not building:** missing front matter or `_posts` misnamed (not plural) or wrong filename date.
- **CSS not applying:** when overriding Minima’s SCSS, keep the three-dash front matter at the **top of your .scss** so Jekyll processes it.

---

## Publish workflow

1. Write locally and preview:

```bash
bundle exec jekyll serve
```

2. Commit & push:

```bash
git add -A
git commit -m "Post: Blog of a blog"
git push
```

---

## What’s next

I’ll keep posting build logs (Jetson Orin Nano, data eng, and AI). Owning the stack means I can ship fast, tweak freely, and keep everything in git.

**Stack:** Jekyll (Minima) + GitHub Pages  
**Why:** Control, portability, Git-first workflow  
**Time to first publish:** An evening; a couple weeks of polish while writing the Jetson post 🛠️

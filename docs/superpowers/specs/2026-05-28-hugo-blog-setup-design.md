# Hugo Blog Setup — Design

**Date:** 2026-05-28
**Owner:** rivik
**Status:** Approved

## Purpose

A personal content hub at `https://rivik.github.io/` that serves two goals:

1. **Discoverability** — make original posts findable via Google.
2. **Backup** — mirror articles published on LinkedIn, dev.to, and iproxy in one durable place under the author's control.

The hub holds two kinds of content side by side:

- **Originals** — first published here. The blog is their canonical source.
- **Reposts** — copies of articles first published elsewhere. The external source remains canonical; the copy here is for archive + indexing.

## Hosting & repo

- GitHub repo: `rivik/rivik.github.io` (renamed from the initial `rivik/rivik-blog`).
- Local directory renamed to match: `~/projects/my/rivik.github.io/`.
- GitHub Pages serves the repo at `https://rivik.github.io/`.
- Pages source: **GitHub Actions** (not the legacy branch-based deploy).
- Posts live at the site root: `https://rivik.github.io/<slug>/` — no `/posts/` or `/blog/` prefix.
- No custom domain at launch. A future custom domain can be added by committing a `CNAME` file in `static/` and pointing DNS at GH Pages.

## Hugo structure

```
rivik.github.io/
├── hugo.toml                  # site config
├── archetypes/
│   └── default.md             # post template (pre-fills frontmatter)
├── content/
│   └── posts/                 # all posts
│       └── *.md
├── static/                    # raw assets (favicon, images, CNAME later)
├── themes/PaperMod/           # git submodule
├── docs/superpowers/specs/    # design docs (not published)
└── .github/workflows/
    └── deploy.yml             # build + publish via Pages Actions
```

- **Theme:** [PaperMod](https://github.com/adityatelange/hugo-PaperMod), installed as a git submodule at `themes/PaperMod`.
- **Single content section:** `posts/`. No other sections at launch.
- **Permalinks:** configured so `content/posts/foo.md` publishes to `/foo/`.
- **Taxonomies:** Hugo defaults — `tags` and `categories`. PaperMod auto-renders index pages at `/tags/` and `/categories/`.
- **Language:** English only. No `i18n/` directory, no `languages` config block.

## hugo.toml (key settings)

```toml
baseURL = "https://rivik.github.io/"
languageCode = "en-us"
title = "rivik"
theme = "PaperMod"

[permalinks]
  posts = "/:slug/"

[params]
  ShowReadingTime = true
  ShowShareButtons = false
  ShowPostNavLinks = true
  ShowCodeCopyButtons = true
  ShowToc = true
  TocOpen = false
  defaultTheme = "auto"

[outputs]
  home = ["HTML", "RSS", "JSON"]   # JSON enables PaperMod search later if wanted
```

(Exact param list refined during implementation; this captures intent.)

## Post frontmatter & repost workflow

Archetype `archetypes/default.md`:

```yaml
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
tags: []
categories: []
summary: ""
canonicalURL: ""   # leave empty for originals; set to source URL for reposts
---
```

Behavior:

- **Originals:** `canonicalURL` empty → PaperMod emits
  `<link rel="canonical" href="https://rivik.github.io/<slug>/">`.
- **Reposts:** set `canonicalURL` to the LinkedIn/dev.to/iproxy URL → PaperMod emits canonical to that URL. The post still appears in the local index, RSS, and sitemap, but search engines treat the external source as authoritative.
- Suggested convention: reposts also carry a `repost` tag (or `Reposts` category) so they can be filtered on the index.

Creating a post:

```bash
hugo new posts/my-article.md
# edit frontmatter, set draft: false when ready
git commit && git push
```

## SEO essentials (all from PaperMod / Hugo defaults)

- `sitemap.xml` at site root.
- RSS at `/index.xml` and per-taxonomy.
- `robots.txt` allowing all crawlers.
- OpenGraph + Twitter card meta tags.
- JSON-LD `Article` schema on post pages.
- After first deploy: submit `https://rivik.github.io/sitemap.xml` to Google Search Console. This is the actual discoverability lever.

## Deployment

`.github/workflows/deploy.yml` based on the official [Hugo GitHub Pages workflow](https://gohugo.io/host-and-deploy/host-on-github-pages/):

- Trigger: push to `main`.
- Steps: checkout (with submodules), setup Hugo extended, `hugo --minify`, upload Pages artifact, deploy.
- GH Pages settings: source = "GitHub Actions".

## Out of scope (deferred until actually needed)

- Analytics.
- Comments.
- Custom domain.
- Custom CSS / theme overrides.
- Search.
- Multilingual content.
- Newsletter / subscription.
- Image optimization pipelines beyond what Hugo does automatically.
- Migration tooling to bulk-import existing LinkedIn/dev.to/iproxy posts. Initial reposts will be added manually.

## Success criteria

- `https://rivik.github.io/` returns a working Hugo site within minutes of `git push` to `main`.
- A new original post created via `hugo new` and published shows correct canonical pointing to itself.
- A repost with `canonicalURL` set to an external URL emits a canonical link to that URL in its HTML head.
- `sitemap.xml` lists all published posts.
- The repo's GitHub Actions workflow passes green.

## Risks & mitigations

- **Renaming the repo breaks the existing clone's remote.** Update the local remote URL after rename (`git remote set-url origin ...`).
- **GH Pages Actions deploy needs Pages enabled with "GitHub Actions" source.** Verify in repo settings before first push, or the workflow's `deploy` step fails.
- **PaperMod as a submodule means `git clone` requires `--recurse-submodules`.** Document this in the eventual README, or pin theme via `hugo module` instead. Submodule is simpler for one solo author and is the path taken here.
- **Duplicate-content SEO penalty on reposts** if `canonicalURL` is forgotten. Mitigated by including the field in the archetype and a tag convention; an optional lint script could check that posts tagged `repost` also have `canonicalURL` set — not built at launch.

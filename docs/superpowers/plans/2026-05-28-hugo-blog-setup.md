# Hugo Blog Setup Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Stand up a Hugo blog at `https://rivik.github.io/` with PaperMod, root-level post URLs, per-post canonical control for reposts, and a GitHub Actions deploy.

**Architecture:** Single-section Hugo site (`content/posts/`) built via `hugo --minify`, deployed to GitHub Pages from `main` via the official Pages Actions workflow. Theme installed as a git submodule. Canonical URLs driven by a `canonicalURL` field in post frontmatter (PaperMod respects it natively).

**Tech Stack:** Hugo (extended) ≥ 0.128, PaperMod theme (git submodule), GitHub Pages with Actions deploy.

**Note on TDD adaptation:** This is config/ops work, not application code. "Tests" here are verification commands run after each change — `hugo` build success, `grep` for expected output in generated HTML, deployed site smoke test.

---

### Task 1: Rename GitHub repo and update local clone

**Files:**
- No files yet — operational only.

- [ ] **Step 1: Rename the GitHub repo**

Run from anywhere:
```bash
gh repo rename rivik.github.io --repo rivik/rivik-blog
```

Expected output includes the new URL `https://github.com/rivik/rivik.github.io`.

- [ ] **Step 2: Verify the local remote still works**

```bash
cd /home/rv/projects/my/rivik-blog
git remote -v
```

GitHub auto-redirects, but we'll fix the URL explicitly to avoid surprises.

- [ ] **Step 3: Update the remote URL**

```bash
cd /home/rv/projects/my/rivik-blog
git remote set-url origin https://github.com/rivik/rivik.github.io.git
git remote -v
```

Expected: both `fetch` and `push` show the new URL.

- [ ] **Step 4: Rename the local directory**

```bash
cd /home/rv/projects/my
mv rivik-blog rivik.github.io
```

From this point on, the working directory is `/home/rv/projects/my/rivik.github.io`.

- [ ] **Step 5: Verify**

```bash
cd /home/rv/projects/my/rivik.github.io
git status
git log --oneline
```

Expected: clean tree, the spec commit `0311b6e` visible.

No commit in this task — rename only.

---

### Task 2: Initialize Hugo site in the existing repo

**Files:**
- Create: `hugo.toml`
- Create: `archetypes/default.md` (Hugo writes this; we overwrite in Task 5)
- Create: `content/`, `data/`, `layouts/`, `static/`, `assets/`, `themes/` (Hugo creates empty dirs; most stay empty for now)

- [ ] **Step 1: Run `hugo new site` in the current directory**

```bash
cd /home/rv/projects/my/rivik.github.io
hugo new site . --force
```

`--force` is needed because `.git` and `docs/` already exist. Hugo will create its scaffolding alongside them.

Expected: "Congratulations! Your new Hugo site was created in ..." — and `hugo.toml` plus several directories now exist.

- [ ] **Step 2: Verify Hugo scaffolding**

```bash
ls -la
```

Expected to see: `hugo.toml`, `archetypes/`, `content/`, `data/`, `layouts/`, `static/`, `assets/`, `themes/`, plus pre-existing `.git/` and `docs/`.

- [ ] **Step 3: Commit the scaffolding**

```bash
git add hugo.toml archetypes content data layouts static assets themes
git status
git commit -m "chore: hugo new site scaffolding"
```

(Some of those dirs may be empty and not get added; that's fine — `git add` will ignore them.)

---

### Task 3: Add PaperMod theme as a git submodule

**Files:**
- Create: `.gitmodules`
- Create: `themes/PaperMod/` (submodule)

- [ ] **Step 1: Add the submodule**

```bash
cd /home/rv/projects/my/rivik.github.io
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

Expected: clones into `themes/PaperMod`, creates `.gitmodules`.

- [ ] **Step 2: Verify the submodule pinned a commit**

```bash
git status
cat .gitmodules
ls themes/PaperMod/layouts | head
```

Expected: `themes/PaperMod` shown as new in `git status` with a specific commit pinned; `layouts/` directory in PaperMod has files like `_default/`, `partials/`, etc.

- [ ] **Step 3: Commit**

```bash
git add .gitmodules themes/PaperMod
git commit -m "chore: add PaperMod theme as submodule"
```

---

### Task 4: Configure hugo.toml

**Files:**
- Modify: `hugo.toml` (replace Hugo's default)

- [ ] **Step 1: Replace `hugo.toml` with the project config**

Overwrite `hugo.toml` with exactly this content:

```toml
baseURL = "https://rivik.github.io/"
languageCode = "en-us"
title = "rivik"
theme = "PaperMod"

enableRobotsTXT = true
buildDrafts = false
buildFuture = false
buildExpired = false

[permalinks]
  posts = "/:slug/"

[outputs]
  home = ["HTML", "RSS", "JSON"]

[params]
  env = "production"
  defaultTheme = "auto"
  ShowReadingTime = true
  ShowShareButtons = false
  ShowPostNavLinks = true
  ShowBreadCrumbs = false
  ShowCodeCopyButtons = true
  ShowToc = true
  TocOpen = false
  ShowRssButtonInSectionTermList = true

[params.homeInfoParams]
  Title = "rivik"
  Content = "Personal hub: originals + reposts from LinkedIn, dev.to, iproxy."

[markup.goldmark.renderer]
  unsafe = true

[markup.highlight]
  noClasses = false
```

- [ ] **Step 2: Build locally to verify config is valid**

```bash
hugo --minify
```

Expected: builds without errors, prints "Total in NNN ms". Some pages will be empty (no posts yet) but that's fine.

- [ ] **Step 3: Verify baseURL appears in generated HTML**

```bash
grep -l "rivik.github.io" public/index.html
```

Expected: file matches (i.e., `https://rivik.github.io/` appears in the rendered HTML).

- [ ] **Step 4: Commit**

```bash
git add hugo.toml
git commit -m "feat: configure hugo site (PaperMod, root permalinks, en-only)"
```

---

### Task 5: Set up .gitignore and post archetype

**Files:**
- Create: `.gitignore`
- Modify: `archetypes/default.md`

- [ ] **Step 1: Create `.gitignore`**

```gitignore
# Hugo build output
/public/
/resources/
.hugo_build.lock

# OS
.DS_Store
Thumbs.db
```

- [ ] **Step 2: Overwrite `archetypes/default.md`**

Replace its contents with:

```markdown
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
tags: []
categories: []
summary: ""
canonicalURL: ""
---

```

- [ ] **Step 3: Verify the archetype works**

```bash
hugo new posts/archetype-smoke-test.md
cat content/posts/archetype-smoke-test.md
```

Expected: file created with title `Archetype Smoke Test`, today's date, `draft: true`, and the `canonicalURL: ""` field present.

- [ ] **Step 4: Delete the smoke-test post**

```bash
rm content/posts/archetype-smoke-test.md
```

(We only used it to verify the archetype; the real first post lands in Task 6.)

- [ ] **Step 5: Commit**

```bash
git add .gitignore archetypes/default.md
git commit -m "feat: add gitignore and post archetype with canonicalURL field"
```

---

### Task 6: Add a first sanity post (original) and verify canonical behavior locally

**Files:**
- Create: `content/posts/hello-world.md`

- [ ] **Step 1: Create the post via archetype**

```bash
hugo new posts/hello-world.md
```

- [ ] **Step 2: Edit `content/posts/hello-world.md` to look like this**

```markdown
---
title: "Hello, World"
date: 2026-05-28
draft: false
tags: ["meta"]
categories: ["Originals"]
summary: "First post on the new hub."
canonicalURL: ""
---

This is the first post on the new hub. The hub will collect originals plus reposts from LinkedIn, dev.to, and iproxy. Reposts will set `canonicalURL` to their source so search engines treat the original as authoritative.
```

- [ ] **Step 3: Build and check the canonical link on the post page**

```bash
hugo --minify
grep -o '<link rel="canonical"[^>]*>' public/hello-world/index.html
```

Expected output (canonical points back to the post itself, since `canonicalURL` is empty):
```
<link rel="canonical" href="https://rivik.github.io/hello-world/">
```

- [ ] **Step 4: Verify the home page lists the post**

```bash
grep -o 'href="/hello-world/"' public/index.html
```

Expected: at least one match.

- [ ] **Step 5: Verify the sitemap includes the post**

```bash
grep '<loc>https://rivik.github.io/hello-world/</loc>' public/sitemap.xml
```

Expected: one match.

- [ ] **Step 6: Smoke-test the repost canonical behavior temporarily**

Edit `content/posts/hello-world.md` and change the `canonicalURL` line to:
```yaml
canonicalURL: "https://example.com/some-external-source"
```

Rebuild and check:
```bash
hugo --minify
grep -o '<link rel="canonical"[^>]*>' public/hello-world/index.html
```

Expected:
```
<link rel="canonical" href="https://example.com/some-external-source">
```

If you don't see that, PaperMod's version doesn't honor `canonicalURL` from frontmatter directly — fall back to using `canonical_url` (snake_case) or override `themes/PaperMod/layouts/partials/head.html` in `layouts/partials/head.html`. Either way, this step confirms the spec's repost mechanism works *before* it matters in production.

- [ ] **Step 7: Revert the canonical change**

Edit `content/posts/hello-world.md` and set `canonicalURL: ""` again.

- [ ] **Step 8: Commit**

```bash
git add content/posts/hello-world.md
git commit -m "feat: add first post (hello-world)"
```

---

### Task 7: Add GitHub Actions deploy workflow

**Files:**
- Create: `.github/workflows/deploy.yml`

- [ ] **Step 1: Create the workflow file**

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.140.2
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Build with Hugo
        env:
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 2: Verify YAML is well-formed**

```bash
python3 -c "import yaml; yaml.safe_load(open('.github/workflows/deploy.yml'))" && echo OK
```

Expected: `OK`.

- [ ] **Step 3: Commit**

```bash
git add .github/workflows/deploy.yml
git commit -m "ci: deploy Hugo site to GitHub Pages via Actions"
```

---

### Task 8: Enable Pages with Actions source

**Files:** None — GitHub settings change.

- [ ] **Step 1: Set Pages source to GitHub Actions**

```bash
gh api -X POST repos/rivik/rivik.github.io/pages -f build_type=workflow \
  || gh api -X PUT repos/rivik/rivik.github.io/pages -f build_type=workflow
```

The first call creates the Pages site if it doesn't exist; the `||` falls back to the update endpoint if it already exists.

Expected: JSON response with `"build_type": "workflow"`.

- [ ] **Step 2: Verify**

```bash
gh api repos/rivik/rivik.github.io/pages | grep -E '"(build_type|html_url|status)"'
```

Expected: `"build_type": "workflow"`, an `html_url` of `https://rivik.github.io/`, and a `status`.

---

### Task 9: Push and verify the deploy

**Files:** None — push + observe.

- [ ] **Step 1: Push to origin**

```bash
cd /home/rv/projects/my/rivik.github.io
git push -u origin main
```

Expected: push succeeds, all commits land on `main`.

- [ ] **Step 2: Watch the Actions run**

```bash
gh run watch
```

(Or `gh run list --limit 1` then `gh run view <id>`.)

Expected: both `build` and `deploy` jobs complete green. Total ~1-2 minutes.

- [ ] **Step 3: Smoke-test the live site**

```bash
curl -sI https://rivik.github.io/ | head -1
curl -s https://rivik.github.io/hello-world/ | grep -o '<link rel="canonical"[^>]*>'
curl -sI https://rivik.github.io/sitemap.xml | head -1
```

Expected:
- `HTTP/2 200` for the home page.
- Canonical link pointing to `https://rivik.github.io/hello-world/`.
- `HTTP/2 200` for the sitemap.

- [ ] **Step 4: Manual checklist for the post-deploy lever**

These are outside automation:

- Submit `https://rivik.github.io/sitemap.xml` to Google Search Console.
- Verify the property in GSC (DNS TXT or HTML file via `static/`).
- Optionally request indexing for `https://rivik.github.io/` once.

Do NOT commit anything in this task — `git status` should be clean.

---

## Done criteria

- `https://rivik.github.io/` returns 200 and shows the `hello-world` post on its home page.
- `hello-world` post page emits `<link rel="canonical" href="https://rivik.github.io/hello-world/">`.
- A locally-tested repost with `canonicalURL: "https://example.com/..."` emits canonical to that URL (verified in Task 6).
- `sitemap.xml` is reachable and lists the post.
- GitHub Actions workflow is green on the latest `main` commit.
- Local working tree is clean; no `public/` or `resources/` tracked.

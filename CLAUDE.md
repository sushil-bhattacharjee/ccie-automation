# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Jekyll-based GitHub Pages site (`_config.yml`: `jekyll-theme-minimal`, kramdown) that publishes CCIE Automation study guides. There is **no build system, no package manager, no test suite, no compile step** — every page is a single self-contained HTML file with inlined CSS and JS. GitHub Pages renders it directly from `main`.

## Local preview

Any of these work; pick whichever is installed:

```bash
# Simplest — iframe loads need an HTTP origin, so don't just open index.html via file://
python3 -m http.server 4000

# Jekyll, if matching GitHub Pages output exactly matters
bundle exec jekyll serve
```

Then open <http://localhost:4000/>.

## Architecture

### The shell: `index.html`

`index.html` is a single-page-app navigation shell. It does not import the guide pages — it loads them into `<iframe id="contentFrame">` via `loadContent(url, element)` (see `index.html:1058`). The sidebar is a hand-maintained tree of `<div class="nav-chapter">` blocks; each leaf is `<div class="nav-item" onclick="loadContent('<file>.html', this)">`.

Status badges in the sidebar:
- `badge-ready` — guide is published; the `<div class="nav-item">` has an `onclick` handler.
- `badge-soon` paired with class `disabled` — placeholder, no `onclick`.

The welcome screen ("guide cards") at `index.html:858+` is a **second** hard-coded list of links. Adding a new guide means updating both the sidebar nav and (optionally) the welcome cards.

### The guides: one self-contained HTML per topic

Every guide in the repo root (`restconf-operation.html`, `git-ops.html`, `K8s-Basics.html`, etc.) follows the same shape: theory/explanation followed by an MCQ quiz rendered from an in-page array.

There are **two co-existing quiz-data conventions** — preserve whichever style the file already uses when editing:

- **Compact (newer)** — `const Q = [{q, s, t, m, opts:[{l, c}], ans:[…], exp}, …]`. Used by `restconf-operation.html`, `K8s-Basics.html`. Fields: `q` question, `s` scenario, `t` YANG tree, `m` multi-select flag, `opts[].l` label, `opts[].c` code/curl block, `ans` correct-index array, `exp` HTML explanation.
- **Verbose (older)** — `const questions = [...]`. Used by most other guides (`git-*`, `nx-os-api`, `xpath-operation`, `netconf-operation`, `yang-when-must`).
- Outlier: `git-cherry-pick.html` defines two arrays, `questions1` and `questions2`.

The render/scoring code is inlined at the bottom of each file and is **not** shared across guides — there is no common JS module. Patterns and selectors (`.q-card`, `.option`, `.exp-tree`, IBM Plex fonts) are duplicated by copy-paste.

### Adding a new guide

1. Drop `<topic>.html` in the repo root (start by copying the closest existing guide so the styling and quiz wiring stay consistent with that family).
2. In `index.html`, add a `<div class="nav-item" onclick="loadContent('<topic>.html', this)">` under the right `<!-- Chapter N: ... -->` block, with `<span class="nav-item-badge badge-ready">Ready</span>`.
3. Optionally add a matching `<div class="guide-card" onclick="loadContentFromCard('<topic>.html')">` to the welcome grid.

### `docs/`

`docs/git_merge_3way.md` and `docs/stash_sync_merge.md` are standalone markdown notes. They are **not** linked from the SPA shell — Jekyll will render them at `/docs/...` if anything references them, but the sidebar bypasses Jekyll entirely by loading raw `.html` files through an iframe.

## Conventions worth preserving

- Self-containment: do not introduce shared CSS/JS files, a build step, or a bundler. The whole point is that each page is droppable, viewable, and committable on its own.
- Each new chapter section in `index.html` is its own `<div class="nav-chapter">` with the same chevron-SVG header pattern — match it exactly so the collapse/expand JS (`toggleChapter`) keeps working.
- Quiz explanations use inline HTML inside the `exp` string (e.g., `<strong>`, `<code>`, `<span class="exp-tree">`). Keep that — the renderers inject it as `innerHTML`.

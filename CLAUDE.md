# CLAUDE.md

This file documents the codebase structure, development workflows, and conventions for AI assistants working on this repository.

## Project Overview

`github-slideshow` is a Jekyll-based presentation tool built on top of [Reveal.js](https://revealjs.com/) (v3.9.2). It is used as a GitHub Learning Lab activity to teach Git and GitHub fundamentals. Slide content is authored in Markdown files; Jekyll compiles them into a Reveal.js-powered HTML slideshow suitable for deployment on GitHub Pages.

- **Static site generator**: Jekyll (Ruby)
- **Presentation framework**: Reveal.js v3.9.2 (via npm)
- **Deployment target**: GitHub Pages
- **License**: MIT

---

## Repository Structure

```
github-slideshow/
├── _config.yml              # Jekyll site configuration and Reveal.js settings
├── _includes/
│   ├── head.html            # HTML <head>: Reveal.js CSS, syntax highlighting
│   ├── script.html          # Reveal.js JS initialization with plugins
│   └── slide.html           # Reusable slide component template
├── _layouts/
│   ├── presentation.html    # Main presentation layout (wraps all slides)
│   ├── print.html           # Print/PDF layout
│   └── slide.html           # Per-slide layout
├── _posts/
│   └── 0000-01-01-intro.md  # First slide (add more slides here)
├── index.html               # Entry point; iterates posts to build the deck
├── Gemfile                  # Ruby dependency declarations
├── Gemfile.lock             # Pinned Ruby dependency versions
├── package-lock.json        # Pinned npm dependency (reveal.js only)
├── script/
│   ├── cibuild              # CI: jekyll build + htmlproofer validation
│   ├── server               # Local dev server
│   ├── setup                # One-time environment setup (macOS / rbenv)
│   └── stage                # Internal staging deployment
├── .editorconfig            # Editor formatting rules
└── .gitignore
```

---

## How Slides Work

Slides are Jekyll posts stored in `_posts/`. Jekyll processes each post and the `presentation.html` layout assembles them into a Reveal.js slide deck.

### Adding a Slide

1. Create a file in `_posts/` using the naming convention `YYYY-MM-DD-<slug>.md`.
2. Add the required front matter:
   ```yaml
   ---
   layout: slide
   title: "My Slide Title"
   ---
   ```
3. Write slide content in Markdown below the front matter.

Posts are rendered in **reverse chronological order** (newest date first). Use the date portion of the filename to control slide order — lower dates appear later, higher dates appear earlier.

### Slide Front Matter Options

The `_includes/slide.html` template supports optional front matter keys:

| Key | Purpose |
|-----|---------|
| `id` | Sets the `id` attribute on the `<section>` element |
| `class` | Adds CSS classes to the slide `<section>` |
| Any `data-*` attribute | Passed through as HTML data attributes on `<section>` |

---

## Jekyll Configuration (`_config.yml`)

Key settings (do not change without understanding downstream effects):

| Setting | Value | Notes |
|---------|-------|-------|
| `markdown` | `kramdown` | Markdown parser |
| `highlighter` | `rouge` | Syntax highlighting |
| `theme` | `moon` (solarized dark variant) | Reveal.js theme |
| Presentation size | 1000×920 px | Set in `reveal` config block |
| `transition` | `linear` | Slide transition animation |
| `backgroundTransition` | `slide` | Background transition |
| `slideNumber` | `"c/t"` | Shows current/total slide numbers |
| `controls` | `false` | Arrow UI controls are hidden |
| `progress` | `true` | Progress bar at bottom |

---

## Development Workflow

### Prerequisites

- Ruby (managed via rbenv recommended)
- Bundler (`gem install bundler`)
- Node.js / npm (for reveal.js, already pinned in `package-lock.json`)

### First-Time Setup

```bash
script/setup
```

This installs Homebrew dependencies, rbenv, bundler, and all gem dependencies. Runs `git submodule update` as well.

### Running Locally

```bash
script/server
# or equivalently:
bundle exec jekyll serve
```

Visit `http://localhost:4000` to preview the slideshow.

Pass additional Jekyll flags by appending them:

```bash
script/server --drafts --livereload
```

### Building for Production

```bash
bundle exec jekyll build
# Output goes to _site/
```

### CI Build & Validation

```bash
script/cibuild
```

This runs:
1. `bundle exec jekyll build --baseurl "."` — builds the site
2. `htmlproofer _site/index.html --empty-alt-ignore` — validates HTML structure and links

**There is no automated test suite.** All correctness checking is done via HTML validation only.

---

## Dependencies

### Ruby (Gemfile)

| Gem | Version | Purpose |
|-----|---------|---------|
| `github-pages` | >= 207 | Jekyll + GitHub Pages ecosystem |
| `html-proofer` | >= 3.13.0 | HTML validation in CI |
| `tzinfo-data` | latest | Timezone support (Windows only) |

Update Ruby gems with `bundle update`.

### JavaScript (package-lock.json)

| Package | Version | Purpose |
|---------|---------|---------|
| `reveal.js` | 3.9.2 | Presentation framework |

The `node_modules/` directory is **checked in** (not gitignored) so no `npm install` step is needed. Update with `npm install reveal.js@<version>`.

---

## Reveal.js Integration

Reveal.js is loaded from `node_modules/reveal.js/` and initialized in `_includes/script.html` with the following plugins:

- **Markdown** (`marked.js` + `markdown.js`) — renders Markdown in slides
- **Highlight** — syntax highlighting for code blocks
- **Notes** — speaker notes support

Configuration lives in `_config.yml` under the `reveal` key and is injected into the `Reveal.initialize()` call via Jekyll's Liquid templating.

---

## Coding Conventions

### Formatting (`.editorconfig`)

| File type | Indentation | Line endings |
|-----------|-------------|-------------|
| Default | Tabs (4-space width) | LF |
| JSON, JS, CSS, SCSS, YAML, HTML | 2 spaces | LF |
| Markdown | 4 spaces | LF |

All files: UTF-8 charset, final newline required.

### Liquid/HTML templates

- Layouts live in `_layouts/`, partials in `_includes/`.
- Prefer `{% include %}` over duplicating HTML.
- Keep Reveal.js config in `_config.yml`, not hardcoded in JS.

### Markdown slides

- One concept per slide; keep slides concise.
- Code blocks should use fenced syntax with language identifiers for proper highlighting.
- Prefer relative links; absolute links break offline use.

---

## Deployment

### GitHub Pages

Push to the `main` branch. GitHub Pages automatically builds and deploys the site. No manual steps required.

### Staging (internal)

```bash
script/stage
```

Builds with a custom `--baseurl`, creates a temporary git repo from `_site/`, and pushes to an internal staging server. Opens the URL in the browser automatically.

---

## Git Conventions

- Feature branches follow the pattern `claude/<description>-<id>` for AI-assisted work.
- Commit messages are imperative, lowercase, and descriptive (e.g., `bump github-pages`, `remove use of circleci`).
- Dependabot handles routine gem/npm dependency updates.
- No merge commits style is enforced; PRs are typically squash-merged.

---

## Known Constraints

- **No JavaScript test suite** — Reveal.js behavior is not unit tested.
- **No linting** — no ESLint, RuboCop, or similar linters are configured.
- **Slide order is date-driven** — the `YYYY-MM-DD` prefix in `_posts/` filenames determines render order; do not use arbitrary dates.
- **Baseurl is commented out** — `baseurl` in `_config.yml` is intentionally commented out; the CI build passes `--baseurl "."` directly.
- **node_modules committed** — `node_modules/reveal.js` is tracked in git; do not add a broad `node_modules/` gitignore entry.

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

### Slide Order

`index.html` iterates posts with `{% for post in site.posts reversed %}`. Jekyll's `site.posts` is reverse chronological by default; `.reversed` flips it back to chronological. The result:

- **Lower dates → appear earlier** (toward the start of the deck)
- **Higher dates → appear later** (toward the end of the deck)
- `0000-01-01-intro.md` (lowest possible date) is therefore the **first** slide

Use the `YYYY-MM-DD` prefix to explicitly control slide order. Do not use arbitrary dates.

### Slide Front Matter Options

The `_includes/slide.html` template reads these front matter keys:

| Key | HTML result | Notes |
|-----|-------------|-------|
| `slide-id` | `id="..."` on `<section>` | Use for anchor links |
| `classes` | Extra CSS classes on `<section>` | Array, e.g. `classes: [big, dark]`; defaults to `slide` class if omitted |
| Any other key | `data-<key>="<value>"` on `<section>` | Passed through as Reveal.js data attributes |

> Note: the default class `slide` is added only when `classes` is empty or absent.

---

## Jekyll Configuration (`_config.yml`)

Key settings (do not change without understanding downstream effects):

| Setting | Value | Notes |
|---------|-------|-------|
| `markdown` | `kramdown` | Markdown parser |
| `highlighter` | `rouge` | Syntax highlighting |
| `solarized.theme` | `dark` | Sets the `<html>` class; controls CSS color scheme |
| Presentation size | 1000×920 px | `reveal.width` / `reveal.height` |
| `reveal.margin` | `0.1` | 10% empty space around content |
| `reveal.minScale` | `0.2` | Minimum scale factor |
| `reveal.maxScale` | `1.5` | Maximum scale factor |
| `reveal.transition` | `linear` | Slide transition animation |
| `reveal.backgroundTransition` | `slide` | Background transition |
| `slideNumber.format` | `"c/t"` | Shows flattened index / total |
| `reveal.controls` | `false` | Arrow UI controls hidden |
| `reveal.progress` | `true` | Progress bar at bottom |
| `reveal.history` | `true` | Pushes slide changes to browser history |
| `reveal.center` | `true` | Vertical centering of slides |
| `reveal.fragments` | `true` | Fragment animations enabled globally |
| `plugins` | `jemoji` | Emoji support in slide content (`:smile:` syntax) |
| `permalink` | `/:title` | Post URL format |

**Kramdown smart quotes** are configured (`lsquo,rsquo,ldquo,rdquo`); straight quotes in Markdown are converted to curly quotes automatically.

**`baseurl` is intentionally commented out.** The CI build passes `--baseurl "."` directly on the command line.

**`exclude` list** contains legacy `reveal.js/*` paths from when Reveal.js was a git submodule. These entries are harmless now that reveal.js lives in `node_modules/`.

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

Installs Homebrew dependencies, rbenv, bundler, and all gem dependencies. Runs `git submodule update` as well.

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
| `tzinfo-data` | latest | Timezone support (Windows/mingw only) |

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

The `presentation.html` layout applies the `solarized.theme` value (default: `dark`) as a class on `<html>`, which the Reveal.js CSS uses to determine the color scheme.

---

## Coding Conventions

### Formatting (`.editorconfig`)

| File type | Indentation | `insert_final_newline` | `trim_trailing_whitespace` |
|-----------|-------------|----------------------|--------------------------|
| Default (all files) | Tabs (4-space width), LF | `false` | `true` |
| JSON, JS, CSS, SCSS, YAML, HTML | 2 spaces, LF | `false` | `true` |
| Markdown | 4 spaces, LF | `true` | `false` |

All files: UTF-8 charset. Note that only Markdown files require a final newline; trailing whitespace is preserved in Markdown (line-break semantics).

### Liquid/HTML templates

- Layouts live in `_layouts/`, partials in `_includes/`.
- Prefer `{% include %}` over duplicating HTML.
- Keep Reveal.js config in `_config.yml`, not hardcoded in JS.
- The `presentation.html` layout wraps each post with `{% include slide.html %}` and a `.page-break` div.
- An optional overview slide is rendered when `site.overview` is truthy (configured in `_config.yml`).

### Markdown slides

- One concept per slide; keep slides concise.
- Code blocks should use fenced syntax with language identifiers for proper highlighting.
- Emoji shortcodes (`:smile:`) work via the `jemoji` Jekyll plugin.
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
- **Slide order is date-driven** — the `YYYY-MM-DD` prefix controls order; lower date = earlier slide. Do not use arbitrary dates.
- **Baseurl is commented out** — `baseurl` in `_config.yml` is intentionally commented out; the CI build passes `--baseurl "."` directly.
- **node_modules committed** — `node_modules/reveal.js` is tracked in git; do not add a broad `node_modules/` gitignore entry.
- **Legacy `exclude` entries** — `_config.yml` still excludes `reveal.js/*` submodule paths; harmless but not worth removing as it matches historical CI behavior.
- **`root = false` in `.editorconfig`** — allows parent-directory EditorConfig files to merge in; no parent config exists in practice.

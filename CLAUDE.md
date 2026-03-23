# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A collection of HTML slide deck presentations hosted on GitHub Pages at `slides.riches.im`. Each presentation is a self-contained HTML file with embedded CSS, JS, and assets. Presentations may include companion pages (e.g. scrollable guides) embedded via iframe.

## Repository Structure

- **Root `index.html`** — Landing page that reads `presentations.json` and renders links to all presentations.
- **`presentations.js`** — Manifest listing all presentation directory names as a `PRESENTATIONS` array. **Must be updated whenever a presentation is added or removed.**
- **Presentation directories** — Named `YYYY-MM-DD Presentation Name`. The first 10 characters are the ISO date, followed by a space, then the name. Each contains an `index.html` and optional image assets.

## Creating a New Presentation

Copy an existing presentation directory, rename it with the new date and name, and update date/event references inside `index.html`. Then add the directory name to `presentations.js`.

## QR Codes

QR code images are generated using the Python `qrcode` library. To generate a new one, create a temporary venv and run:

```bash
python3 -m venv /tmp/qr-venv && /tmp/qr-venv/bin/pip install 'qrcode[pil]'
/tmp/qr-venv/bin/python3 -c "
import qrcode
qr = qrcode.QRCode(version=1, error_correction=qrcode.constants.ERROR_CORRECT_H, box_size=20, border=2)
qr.add_data('https://example.com')
qr.make(fit=True)
img = qr.make_image(fill_color='black', back_color='white')
img.save('output-qr.png')
"
```

Changing a QR code's target URL requires generating a new image — the URL is encoded in the image pixels, not in HTML.

## Slide System

Each presentation's `index.html` contains:

- **Slides**: `<div class="slide">` elements. Only `.slide.active` is visible. Variants: `.slide.dark` (dark background), `.slide.center` (centered content).
- **Navigation**: Arrow keys / Space for next, ArrowLeft for prev, `f` for fullscreen (guarded with `!e.metaKey&&!e.ctrlKey` so Cmd+F/Ctrl+F still opens browser search). Click buttons also provided.
- **Reveal mechanic**: Slides with `data-reveal` attribute show content progressively — first keypress adds `.revealed` class (animates in `.problem-fix` elements), second keypress advances to next slide.
- **Slide comments**: Each slide is preceded by `<!-- SLIDE: DESCRIPTION -->` comments (no numbers). When a user refers to a slide by number, count `<div class="slide">` elements from the top to identify it.
- **Slide counter**: The `#counter` element and `total` variable in JS are computed dynamically from `document.querySelectorAll('.slide')`.

## Design System

**Shared CSS** (`shared.css`): When a presentation includes companion pages (e.g. a scrollable guide loaded via iframe), shared design tokens and base components live in `shared.css` within the presentation directory. Both the presentation and companion pages link to it. Contains: CSS reset, `:root` variables, `.accent`, `.mono`, `.divider`, `.card` (base). Each page overrides only what differs (e.g. padding, margins).

**CSS Variables** (`:root`): `--purple: #9200E1`, `--bg: #FEFEFE`, `--text: #0A0A0A`, `--red: #E83E3E`, `--green: #16A34A`, `--font: 'Inter'`, `--mono: 'JetBrains Mono'`.

**Key components**: `.tag` (section label with dot), `.card` / `.card-grid` (bordered containers), `.stack` / `.stack-layer` (architecture diagrams), `.bar-chart` / `.bar-fill` (horizontal bars), `.compare-table` (comparison grids), `.problem-layout` (two-column problem/solution with CSS graphics), `.two-col` (generic two-column grid).

**Typography**: `h1` 62px/300wt, `h2` 44px/300wt, `h3` 22px/500wt. Bold via `<strong>` (700wt). Purple highlights via `.accent`. Subtitles via `.subtitle`.

**Visual effects**: Radial dot grid background on every slide (`::before`), purple glow (`::after`), `fadeUp` animation with staggered delays on slide children.

## Companion Pages (iframe slides)

Scrollable content (e.g. guides, reference pages) can be embedded as iframe slides: `<div class="slide" style="padding:0;overflow:hidden"><iframe src="page.html" ...></div>`. Keyboard forwarding uses `postMessage`: the companion page detects it's in an iframe and posts `{type:'slideKey', key}` to the parent, which listens via `window.addEventListener('message')`. The parent's 0G logo is automatically hidden on iframe slides.

## Deployment

GitHub Pages from `main` branch. Custom domain via `CNAME` file (`slides.riches.im`). Commit signing is configured but may require GPG passphrase — use `--no-gpg-sign` if signing fails.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-page law firm website for Maroun Mansour Law Office. Entirely self-contained in one file: `index.html` (~3 000 lines). No build step, no dependencies, no package.json.

**Live site:** https://finalboss3000.github.io/maroun-mansour/  
**Repo:** https://github.com/FinalBoss3000/maroun-mansour  
Deployed via **Cloudflare Pages** (connected to this repo, builds on push to `main`). Security headers are served from the `_headers` file at the repo root — Cloudflare Pages applies it automatically; it has no effect on local `npx serve` previews or plain GitHub Pages.

## Running locally

```bash
# Start preview server (port 3456)
npx serve -p 3456 -s .
```

Or use the Claude Code preview tool — `launch.json` is already configured (`maroun-mansour-website`).

## Architecture — index.html layout

The file is structured top-to-bottom in three blocks:

### 1. `<style>` (lines ~18–1320)
CSS sections in order, separated by `/* ════ */` banners:
- **Design tokens** — all colours (`--c-*`), fonts (`--f-*`), spacing (`--sp-*`), radii (`--r-*`)
- **Shared components** — `.btn`, `.eyebrow`, `.sec-heading`, `.container`
- **Section CSS** — NAV → HERO → TRUST → PRACTICE → ABOUT → APPROACH → CONTACT → FOOTER
- **Modal CSS** — `.pmodal` (practice area) then `.lmodal` (legal/policy)
- **Cookie banner** — `.cookie-bar`
- **Accessibility widget** — `.a11y-toggle`, `.a11y-panel`, `.a11y-feat`, plus all `html.a11y-*` override classes
- **RTL support** — all `[dir="rtl"]` overrides at the bottom; use `:lang(he)` / `:lang(ar)` for font-specific overrides
- **Responsive** — a single `@media` block at the very end

### 2. `<body>` HTML (lines ~1325–2065)
Sections in visual order: NAV → `<main>` (HERO, TRUST BAR, PRACTICE AREAS, ABOUT, APPROACH, CONTACT) → FOOTER → modals → widgets.

Fixed/overlay elements appended after `</footer>`:
- `#pmodal` — practice area detail modal
- `.a11y-guide-line`, `.a11y-mask-*`, `#a11y-toggle`, `#a11y-overlay`, `#a11y-panel` — accessibility widget
- `#cookie-bar` — cookie consent banner
- `#lmodal` — legal modal (Accessibility Statement + Cookie Policy)

### 3. `<script>` (lines ~2069–3054)
Single inline script, sections in order:
1. **`T` object** — translation map `{ en: {...}, he: {...}, ar: {...} }` with ~100 keys each
2. **`setLang(lang)`** — switches language, sets `html.lang` + `html.dir` (RTL for `he`/`ar`), updates `<title>` and meta description, persists to `localStorage`
3. **Scroll reveal** — IntersectionObserver adds `.in` to `.rev` elements
4. **Nav scroll** — adds `.scrolled` to nav after 60 px
5. **Mobile menu** — `openMenu()` / `closeMenu()`
6. **Contact form** — validates then POSTs (`mode:'no-cors'`) to a Google Apps Script web app that emails the firm
7. **Practice modal IIFE** — wires `.pcard` clicks → `#pmodal`, re-translates on language switch
8. **Cookie consent IIFE** — shows `#cookie-bar` if `localStorage['cookie-consent']` unset
9. **`LEGAL` object** — `{ en, he, ar }` × `{ accessibility, cookies }` with full HTML body strings
10. **`openLegal(type)`** — populates and opens `#lmodal`
11. **Accessibility widget IIFE** — 18 toggleable features, saves/restores `localStorage['a11y-prefs']` JSON

## Key patterns

**Adding/changing text:** Every user-visible string uses `data-i18n="key"` (HTML content) or `data-i18n-placeholder="key"` (input placeholders). Add the key to all three language objects in `T` before using it in HTML.

**Adding a new section:** Add CSS in the `<style>` block (keep the `/* ════ */` banner pattern), HTML in `<main>`, and `data-i18n` attributes on all text nodes. Add `.rev` class to elements that should animate in on scroll.

**RTL fonts:** Hebrew headings use `Frank Ruhl Libre`; Arabic headings use `Noto Naskh Arabic`. Both are loaded from Google Fonts. Use `:lang(he)` / `:lang(ar)` selectors — not `[dir="rtl"]` — for font overrides, because Arabic and Hebrew need different typefaces.

**Accessibility overrides:** Each feature adds a class to `<html>` (e.g. `html.a11y-greyscale`). CSS rules targeting those classes live in the "Applied accessibility overrides" block inside `<style>`. The JS feature→class map is `FEATS` inside the widget IIFE.

**Form submissions** go to a Google Apps Script web app (`https://script.google.com/macros/s/.../exec`, `APPS_SCRIPT_URL` constant in the contact-form handler). The POST uses `mode:'no-cors'`, so the response can't be read — success is assumed if `fetch` doesn't throw. The Apps Script project (deployed under the firm's Google account) receives the form fields and emails them on. A `honeypot` hidden field silently drops bots.

## Assets

| File | Purpose |
|------|---------|
| `logo.svg` | Primary logo (also used as favicon) |
| `logo.avif` | AVIF version of logo for `<picture>` element |

No other external assets — all images in the site are loaded from the Wix CDN (if any practice area photos are used) or are inline SVGs.

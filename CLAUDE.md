# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal site for Santosh Rajput — pure static HTML/CSS/JS with **no build step**, no package manager, no framework. Deployed via GitHub Pages at https://rajputsantosh.github.io/santosh-rajput-site/. To "develop" you just open the HTML file in a browser; to "deploy" you commit and push to `main`.

There are no tests, linters, or scripts. The only tooling is `git`.

## Pages

Four shipped pages, each fully self-contained (own `<style>`, own `<script>`, own nav, own JSON-LD schema):

- `index.html` — the main long-scroll site. ~2300 lines, all CSS and JS inline. Contains the hero, career circuit, world map, speedometer KPIs, wins, AI stack, testimonials, contact, modal system.
- `operator.html`, `board.html`, `speaking.html` — buyer-specific landing pages linked from the home nav.
- `404.html` — GitHub Pages fallback.
- `sitemap.xml` lists the four canonical URLs above; keep it in sync when adding pages.

Internal-only files (do not link from public nav):
- `logo-viewer.html` — local grid for previewing `logos/imageNN.png` while picking client marks.
- `tweaks-panel.jsx` — a reusable React/Babel "tweaks panel" component for prototyping. Not currently loaded by any HTML page; included as scaffolding for ad-hoc experiments.
- `chats/chat1.md` — historical design/decision transcript, not shipped.

## Architecture conventions

**Single-file pages.** Each `.html` is a complete document with its CSS in one `<style>` block and its JS in one `<script>` block at the bottom. Don't extract shared stylesheets or JS modules — that's intentional. When the same nav/card/button needs to exist on two pages, the CSS is duplicated. Match the existing copy when editing.

**Design tokens live in `:root`.** Every page redeclares the same custom-property palette and font stack:

```
--bg / --bg2 / --bg3   /* charcoal layers */
--ink / --ink2 / --ink3 /* foreground text tiers */
--red / --red2          /* accent — currently teal (#5fb3a3 / #7cc7b8), historical name */
--cond  Barlow Condensed (display)
--sans  Inter (body)
--mono  Share Tech Mono (eyebrows, labels)
```

The `--red*` variables are **named for legacy reasons** — the palette was switched from red+black to charcoal+teal (commit 451922f). Keep the variable names; just change the values if the palette shifts again. When updating the palette, update **all four pages plus `logo-viewer.html`** so nothing drifts.

**F1 / racing telemetry aesthetic.** Uppercase Barlow Condensed for titles, mono eyebrows prefixed with `//`, clip-path angle-cut buttons (`clip-path: polygon(8px 0, 100% 0, ...)`), grid background lines, accent ring/dot markers. Stay inside that vocabulary — don't introduce gradients, drop shadows, or rounded buttons that break the look.

**`index.html` interactive systems** (bottom `<script>` block, ~line 1943 onward):
- `.fade-up` elements get `.vis` via an IntersectionObserver — apply this class to anything that should animate in on scroll.
- `fillOnEnter(wrapperSelector, fillSelector)` drives staggered `width: data-pct%` animations for `.trajectory-wrap .tr-bar-fill` and `.pit-card .pit-meter-fill`. Use the same `data-pct="…"` pattern for new bar charts.
- `countUpEl` animates numbers matched by the `numSelectors` string. To make a new number count up, add its selector to that list (or one of the existing classes) and put the final value as text content.
- `drawSpeedo` / `animSpeedo` render the four KPI gauges on canvases `speedo1..4` and update `#sv1..#sv4`. Targets are wired in the IntersectionObserver around line 2167.
- World map (line ~2180) loads `vendor/countries-110m.json` via `fetch`, then dynamically injects `vendor/topojson-client.min.js`. Pins, arcs and tooltips are built from the `locations` array — edit that array to change geo coverage.
- Modal system: `modalData` (line ~2026) maps a `data-modal` key to `{eyebrow, title, big, sub, rows, note}`. Add a card with `data-modal="newKey"` and a matching entry to open in the shared overlay (`#modal-overlay`).

**SEO/social per page.** Every page carries its own `<title>`, meta description, canonical, OG/Twitter tags, and `application/ld+json` block (typically `Person` on index, `Service` on the landing pages). When changing positioning copy, update **all** of these together — commit 319afdf shows the expected scope of such a change.

**Assets.**
- `logos/imageNN.{png,jpg,jpeg}` — client logos referenced by index numbers in the clients grid. New logos go here and are picked by editing `index.html`; preview via `logo-viewer.html` (extend its grid when you add files).
- `uploads/` — resume, og-card, profile photo, pasted screenshots. Linked from pages by exact filename, so renaming a file means updating every HTML reference.
- `vendor/` — pinned third-party files for the world map. Do not replace with a CDN URL; offline-friendly is a goal.

## Working in this repo

- Open the page directly: `xdg-open index.html` (or just drag into a browser). No dev server needed, though `python3 -m http.server` works if you want `fetch('vendor/...')` to be served over HTTP rather than `file://`.
- After visual edits, scroll through the **whole** affected page in a browser — the inline JS is observer-driven and bugs usually surface only when a specific section enters the viewport.
- Commit messages here are descriptive and action-oriented (see `git log`). Examples: `"Trim plan applied: Voices 7->3, Wins 12->7"`, `"Switch live palette: red+black -> charcoal + lighter teal across all 4 pages"`. Match that style: imperative summary, mention which pages/sections were touched, include before -> after for numeric/copy swaps.
- The site is deployed by GitHub Pages from the default branch. Don't introduce a build step, transpiler, or `node_modules` without an explicit request — the no-build-step property is a feature.

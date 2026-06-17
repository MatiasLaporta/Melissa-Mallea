# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page commercial proposal ("Propuesta a Melissa Mallea") for the client Melissa Mallea (Sales Specialist Hunter B2B at Global66), built as a static Astro site styled like an executive dashboard. It is deployed to GitHub Pages at the custom domain **melissamallea.matiaslaporta.com**. SEO is intentionally omitted — this is a private proposal, not an indexed site.

> Note: this project was forked from a prior "Propuesta Kargo-Log" proposal. The dark-dashboard layout, tab engine, and animated node background are reused; the showcase tab and client-brand palette from that proposal were removed.

## Commands

```bash
npm run dev        # dev server (auto-picks next free port: 4321, 4322, ...)
npm run build      # static build to dist/
npm run preview    # serve the built dist/
```

There are no tests, linters, or a test runner configured. Validation = `npm run build` passing.

## Stack

- **Astro 5** + **Tailwind CSS v4** via the official `@tailwindcss/vite` plugin (configured in `astro.config.mjs`). There is **no `tailwind.config.js`** — Tailwind v4 is config-less here; custom colors use arbitrary values (`bg-[#0f172a]`) and any custom animations live in a `<style is:global>` block. Tailwind is imported through `src/styles/global.css` (`@import "tailwindcss";`), which `index.astro` imports.
- Plain vanilla `<script>` for all interactivity. No framework components, no client-side router.

## Architecture

**The entire site is one file: `src/pages/index.astro`.** It is served at the site root `/` (there is no separate index/redirect page). Structure within the file:

1. **Frontmatter data arrays** (top of file) — `tabs`, `modulos`, `precios`, `cronograma`. Edit copy/pricing here, not in the markup; the markup `.map()`s over these.
2. **`<style is:global>`** — tab-active states and `<details>` marker hiding.
3. **Sidebar** (`lg:` fixed) and **mobile header** (`lg:hidden`, hamburger menu) — both render the same `tabs` array; their buttons share `data-tab` and the `nav-side`/`nav-top` classes.
4. **Three tab panels** — `<section data-panel="...">`: `diagnostico`, `soluciones`, `inversion`.
5. **Trailing `<script>`** — the tab engine.

### Tab system (vanilla JS)
`activate(id)` is the core function: toggles `.hidden` on every `[data-panel]`, sets `.tab-active` on every matching `[data-tab]` button (sidebar + mobile menu + in-content buttons all use `data-tab`), closes the mobile menu, and scrolls to top. The default tab is `diagnostico`.

### Animated node background (constellation)
A single fixed `#bg-nodes` canvas reproduces the particle network from matiaslaporta.com (color `#00E5FF`). It is shown **only on the tabs in `NODE_TABS`** (`diagnostico` + `inversion`) and its `requestAnimationFrame` is cancelled otherwise. Key constants in the IIFE: `COUNT_DESKTOP = 46`, `COUNT_MOBILE = 20`, `LINK_DIST`, `MOUSE_DIST`.

**Gotcha:** `<canvas>` is a replaced element, so `position:fixed; inset:0` does NOT stretch it — it would size to the intrinsic 300×150. The canvas is therefore sized explicitly in JS (`resize()` sets `style.width/height/left` from `window.innerWidth` minus the 288px sidebar) rather than relying on CSS. Any new canvas must do the same.

## Color system

Dark `#0f172a` / `#0b0f19`, accents **cyan** (`cyan-400`/`#22d3ee`) and **yellow** (`yellow-400`). Headings use yellow highlights; CTAs are yellow and link to the client's Google Calendar.

## Pricing (tab 3 · Inversión)

Prices are in Chilean pesos (CLP), shown with a dot thousands separator (e.g. `$92.000`). Stored in the `precios` array. Taking all 3 modules applies a 30% discount (the highlighted "Paquete Completo · 3 Módulos" card). No currency conversion is shown.

## Deployment

Push to `main` → `.github/workflows/deploy.yml` (uses `withastro/action`) builds and publishes to GitHub Pages. `public/CNAME` holds the custom domain and `site` is set in `astro.config.mjs`; both must stay in sync with the domain.

**Asset naming gotcha:** GitHub Pages serves on Linux (case-sensitive). Files in `public/` referenced from markup must use **lowercase, no-spaces** names (e.g. `logo-ml-inv.png`, not `Logo ML.png`) or they 404 in production while working locally on Windows. The git account for this repo is `MatiasLaporta`.

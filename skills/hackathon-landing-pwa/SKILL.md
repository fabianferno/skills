---
name: hackathon-landing-pwa
description: >-
  Landing page and app shell polish for Next.js hackathon projects: Lenis smooth scroll, SEO and social images
  (favicon set, web manifest, Open Graph/Twitter banner from a seed asset), global selection styling, PWA basics,
  conversion-oriented copy frameworks, and heading/body font pairing. Use when building or refining a marketing
  landing page, app metadata, installable PWA, favicons, or scroll/typography polish alongside hackathon-app-dev.
---

# Hackathon landing page, SEO, and PWA

## When to use

Apply when shipping or tightening the **public landing** and **browser chrome** for a Next.js app: smooth scrolling, **favicons and manifest**, **OG/Twitter** previews, **PWA** affordances, **brand-colored text selection**, and **copy structure** that converts.

Pair with **[hackathon-app-dev](../hackathon-app-dev/SKILL.md)** for the core stack; use **`frontend-design`** (or equivalent) for visual direction on the hero and layout.

---

## Smooth scroll (Lenis + React)

1. Install: `pnpm add lenis` ([`lenis/react`](https://github.com/darkroomengineering/lenis/blob/main/packages/react/README.md)).
2. Wrap the app (or layout) with `<ReactLenis root />` from `lenis/react`.
3. If the project already uses **Framer Motion**, wire Lenis `raf` to Motion’s `frame` loop per the [Lenis React README — Framer Motion integration](https://github.com/darkroomengineering/lenis/blob/main/packages/react/README.md) (`autoRaf: false`, `frame.update` / `cancelFrame`).
4. Tune easing/duration via Lenis `options` only if defaults feel wrong on the target devices.

---

## SEO, icons, and social preview

### Seed asset

1. **Ask the user for a seed image** (logo, mark, or hero graphic) as the single source of truth for the brand mark.
2. **Vector**: derive or simplify a **SVG** suitable for small sizes (favicon clarity). Use tracing/simplification in a design tool or an automated vectorizer if needed; avoid illegible detail at 16×16.
3. **Rasters**: export or generate from the seed/SVG at least:
   - `android-chrome-192x192.png`
   - `android-chrome-512x512.png`
   - `apple-touch-icon.png`
   - `favicon-16x16.png`
   - `favicon-32x32.png`
   - `favicon.ico` (multi-size ICO from the same mark)
   - `site.webmanifest` (name, short_name, theme/background colors, `icons` pointing at the above)
4. **Open Graph / Twitter banner**: produce **`site-banner.png`** (typical aspect **1200×630** or current platform recommendation). **After the landing renders in the browser**, use the **Cursor browser tools** to **capture a screenshot** of the landing (above-the-fold or full page per design) and save/export as `site-banner.png` for consistent social previews, unless the user supplies a designed asset instead.
5. Wire **Next.js App Router `metadata`** (or `metadata` API): `title`, `description`, `openGraph.images`, `twitter.card` / `twitter.images`, `icons`, `manifest` path. Remove reliance on the **default `app/favicon.ico`** from `create-next-app` once custom icons live in `public/` and metadata points to them.

### Checklist (files in `public/` unless the project standardizes elsewhere)

- [ ] `android-chrome-192x192.png`, `android-chrome-512x512.png`
- [ ] `apple-touch-icon.png`
- [ ] `favicon-16x16.png`, `favicon-32x32.png`, `favicon.ico`
- [ ] `site-banner.png` (OG/Twitter)
- [ ] `site.webmanifest`
- [ ] Metadata + no duplicate stale default favicon

---

## Global CSS: selection color

In global styles (e.g. `app/globals.css`), set `::selection` (and `::-moz-selection`) **foreground/background** to match the **brand theme** (CSS variables from Tailwind/shadcn theme tokens are ideal). Remove or override any default that clashes with the palette.

---

## PWA (optimize the web app)

- **`site.webmanifest`**: `name`, `short_name`, `start_url`, `display` (`standalone` or `browser`), `theme_color`, `background_color`, icon entries.
- **Service worker**: add a maintained solution compatible with the project’s Next.js version (e.g. community **Serwist** / **next-pwa** patterns or the approach in current docs—verify against Next release notes; do not ship a dead config).
- **Installability**: HTTPS in production, maskable icons where applicable, and a **clear `start_url`**.
- **Scope**: for a hackathon, prefer a **minimal** SW (cache shell + offline fallback) over over-caching dynamic API routes.

---

## Copy and typography (reference)

- **Conversion copy**: WHO/WHY/WHAT headlines, CTA → **CTV** (benefit-led), **PAS**, benefits over features, social proof, visual story. Full patterns and examples: [reference.md](reference.md#landing-page-copywriting).
- **Fonts**: humanist/geometric **display** for headings + **Inter-class** body from Google Fonts; optional display fonts (incl. DaFont) and base list: [reference.md](reference.md#fonts).

---

## Handoff

- Core stack, auth, and MCPs: **[hackathon-app-dev](../hackathon-app-dev/SKILL.md)** and [hackathon-app-dev/reference.md](../hackathon-app-dev/reference.md).

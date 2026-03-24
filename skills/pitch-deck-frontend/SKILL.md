---
name: pitch-deck-frontend
description: >
  Use when building a web-based pitch deck, investor presentation, or slide-based page in a frontend app.
  Triggers on requests like "create a pitch page", "build a presentation", "make slides for investors",
  "pitch deck in the app", or any full-screen scrollable slide layout. Do not use emojis; use lucide-react
  icons for visual accents when needed.
---

# Pitch Deck Frontend

Build polished, full-screen slide presentations inside web apps. Covers layout architecture, typography scale, color discipline, content rules, and interactive navigation.

## No emojis — use Lucide React

**Do not use emojis** anywhere in the pitch UI or slide copy (headings, bullets, labels, navigation hints, “live” indicators, social proof, etc.). They read informal, render inconsistently across OS/fonts, and clash with a tight investor deck.

When you need a visual beat (checkmarks, arrows, category marks, “pulse” indicators, external links), use **`lucide-react`** icons with the deck’s **accent** color and shared size classes (e.g. `h-4 w-4` or `h-5 w-5`). Prefer outline-style icons that match the monospace / technical tone unless the slide pattern explicitly calls for filled marks.

## Architecture

Use a **snap-scroll container** with one full-viewport section per slide.

```
Fixed overlay (z-50):  slide indicator dots, arrows, slide counter, keyboard hint
Scroll container:      snap-y snap-mandatory, overflow-y-auto, h-screen
  └─ Slide wrappers:   snap-start, h-screen, flex-center
       └─ Content:      max-w-[1200px], px-20 (80-100px margins)
```

**Navigation layers:**
- Keyboard: ArrowRight/Down/Space = next, ArrowLeft/Up = prev
- Click: left/right arrow buttons (hide at boundaries)
- Dot indicator: fixed right edge, active dot elongated + accent color
- Scroll: IntersectionObserver syncs dot state to manual scrolling
- Slide counter: bottom-right, monospace, muted (`01 / 09`)

**State management:** single `currentSlide` index. `isScrolling` ref prevents observer from fighting programmatic `scrollIntoView`.

## Typography Scale (1920x1080 baseline)

| Element | Tailwind | Rule |
|---------|----------|------|
| Section label | `text-sm font-mono tracking-widest uppercase` | Accent color, e.g. `01 — Section Name` |
| Slide heading | `text-5xl md:text-6xl` + display font | Max 6 words |
| Key stat | `text-6xl` + display font | One per card, accent color |
| Body text | `text-base` or `text-sm` | Max 6 bullets per slide |
| Caption/mono | `text-xs font-mono` | Sources, addresses, tags |

**Font pairing:** display/bold font for headings + stats, sans-serif for body, monospace for labels/code/data.

## The 1-6-6 Rule

- **1** idea per slide
- **6** words max per bullet
- **6** bullets max per slide

If content overflows, split into more slides. Never cram.

## Color Discipline

| Element | Rule |
|---------|------|
| Background | Single dark color (`#0a0a0a`, navy, charcoal) — commit to one |
| Accent | ONE brand color for all highlights (stats, labels, active states, icons) |
| Body text | `white/50` to `white/60` — never full white for body |
| Headings | Pure white |
| Muted/captions | `white/30` to `white/40` |
| Cards | `bg-white/[0.02]` + `border-white/10` — barely visible surface |
| Accent cards | `bg-accent/[0.03]` + `border-accent/20` — subtle glow |
| Negative | Red-400 for pain points, competitor failures |
| Positive | Accent color for your wins, checkmarks |

**Never:** gradients on text, neon colors, more than 3 total colors, white backgrounds mixed with dark.

## Slide Patterns

### Title Slide
- Logo/icon (animated entrance: scale + fade)
- Product name in display font (staggered fade-in)
- Tagline with accent-colored key metric
- Pill badges for positioning (e.g. "Polkadot Hub", "Orbital AMM")
- Bounce chevron-down at bottom

### Problem/Pain Slide
- Quote or bold claim as heading
- Icon + label + description pairs (red-tinted icons for pain)
- Big stat card with visual bar showing the problem
- Italic closing line

### Why Now / Market Timing
- Numbered card grid (01, 02, 03)
- Each card: number → title → detail → highlighted takeaway
- Cards hover to accent border
- Closing mono-text summary line

### Product / Solution
- Heading with accent-colored key word
- Descriptive paragraph with inline mono-styled formulas
- Blockquote card for the memorable pitch line
- 3-column stat cards: icon → big number → label
- Closing differentiator statement

### Comparison / Competitive
- Grid table: row per competitor, columns per feature
- Accent checkmarks for your product, red × for others
- Your row: accent color, bold, larger text
- Keep to 3-4 competitors max

### Technical Architecture
- Card grid: 2×2 for 4 subsystems
- Primary card (key differentiator) gets accent border/bg
- Pill badges for tech components
- Flow diagram for execution path: boxes + arrows
- Bottom banner for the key insight quote

### Demo / Product Tour
- Numbered step cards (01-04) in 2×2 grid
- Step: number → title → description
- "What it proves" section: checkmark list in 4 columns
- Live indicator: pulsing dot + chain ID / environment

### Traction + Roadmap (combined)
- Split layout: shipped (left, accent bg) vs planned (right, neutral)
- Shipped: big number + label pairs
- Planned: timeline dots with connector lines, first dot filled

### Team
- Centered layout, narrower max-width
- Avatar circle with initials (accent border)
- Name + role (accent mono text)
- Checkmark credential list

### References / Closing
- 2-column: contracts table + sources list
- Truncated addresses with monospace
- External link icons for references
- Author/detail subtext per reference

## Card Design System

```
Neutral card:   rounded-2xl border-white/10 bg-white/[0.02] p-8
Accent card:    rounded-2xl border-accent/20 bg-accent/[0.03] p-8
Hover effect:   hover:border-accent/30 transition-colors
Stat card:      icon (accent, mb-2) → big text (display font) → label (white/40)
Step card:      mono number (accent) → title (white, mt-3) → desc (white/40, mt-2)
Tag pill:       px-3 py-1 rounded-full bg-white/5 border-white/10 text-xs font-mono
```

## Content Mapping

Map source document sections 1:1 to slides. Don't invent extra slides or merge sections. If the source has 8 sections, build 8 slides + title = 9 total.

**Section label format:** `01 — Section Name` in accent mono text above heading.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Too many slides (15+) | Map 1:1 to source sections, max ~12 |
| Wall of text on slide | Apply 1-6-6 rule, split if needed |
| Multiple ideas per slide | One concept, one visual focus |
| Inconsistent card styles | Reuse the same 2-3 card variants |
| Full white body text | Use white/50-60, reserve white for headings |
| Missing keyboard nav | Always add arrow key + spacebar handling |
| No scroll sync | IntersectionObserver to sync dots with manual scroll |
| Centering body text | Left-align body, only center title slide |
| Charts without labels | Label directly on chart, cite sources |
| Pie charts | Never. Use bar charts or big stat numbers |
| Emojis in slides or chrome | **Forbidden** — replace with **lucide-react** icons + text |

---

## Additional resources

- Layout skeleton, observer/navigation patterns, a11y, token checklist: [reference.md](reference.md)

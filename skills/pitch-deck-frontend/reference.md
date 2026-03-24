# Pitch deck frontend — reference

Implementation-oriented companion to [SKILL.md](SKILL.md): DOM structure, interaction patterns, a11y, and copy rules you can paste into a React/Next app.

---

## Layout skeleton (conceptual)

```text
<div class="fixed inset-0 z-50 pointer-events-none">  <!-- overlay: pointer-events only on controls -->
  <!-- dots, arrows, optional progress -->
</div>
<div
  ref={scrollRef}
  class="h-screen overflow-y-auto snap-y snap-mandatory"
  role="region"
  aria-label="Presentation slides"
>
  {slides.map((slide, i) => (
    <section
      key={i}
      id={`slide-${i}`}
      data-slide-index={i}
      class="snap-start min-h-screen flex flex-col justify-center px-8 md:px-20"
      aria-label={`Slide ${i + 1} of ${slides.length}`}
    >
      <div class="w-full max-w-[1200px] mx-auto">{/* content */}</div>
    </section>
  ))}
</div>
```

Use **`min-h-screen`** if any slide can grow (e.g. small viewports); keep **`snap-start`** on each slide wrapper.

---

## State and navigation

- **`currentSlide`**: integer index, 0-based.
- **`isScrolling` ref**: set `true` before `scrollIntoView` / animated scroll; ignore `IntersectionObserver` callbacks while true; clear after `scrollend` or a short timeout fallback.
- **Keyboard**: `ArrowRight` / `ArrowDown` / `Space` → next; `ArrowLeft` / `ArrowUp` → prev; optional `Home` / `End` for first/last.
- **Dots**: `button` elements with `aria-current={i === currentSlide}` on the active dot.
- **Counter**: e.g. `String(currentSlide + 1).padStart(2, '0')` + ` / ` + total.

---

## IntersectionObserver (sync scroll ↔ dots)

Rough pattern:

```typescript
const observer = new IntersectionObserver(
  (entries) => {
    if (isScrolling.current) return;
    const visible = entries
      .filter((e) => e.isIntersecting)
      .sort((a, b) => b.intersectionRatio - a.intersectionRatio)[0];
    const idx = visible?.target.getAttribute('data-slide-index');
    if (idx != null) setCurrentSlide(Number(idx));
  },
  { root: scrollRef.current, threshold: [0.25, 0.5, 0.75] }
);
// observe each section; disconnect on unmount
```

Tune **threshold** / **rootMargin** so manual flick-scroll still picks a single “primary” slide.

---

## Typography (Tailwind recap)

| Role | Classes |
|------|---------|
| Section label | `text-sm font-mono tracking-widest uppercase text-accent` |
| Slide title | `text-5xl md:text-6xl font-bold` + display font family |
| Hero stat | `text-6xl` + display + accent color |
| Body | `text-base text-white/60` (or `text-sm`) |
| Caption | `text-xs font-mono text-white/40` |

**1-6-6 rule**: one idea per slide; ≤6 words per bullet; ≤6 bullets. Split slides instead of shrinking type.

---

## Color tokens (recommended)

Define once and reuse:

```css
:root {
  --deck-bg: #0a0a0a;
  --deck-accent: /* single brand hue */;
  --deck-text: rgba(255, 255, 255, 0.55);
  --deck-heading: #ffffff;
  --deck-muted: rgba(255, 255, 255, 0.35);
}
```

- **One** dark background, **one** accent, body **never** pure white.
- Cards: subtle `bg-white/[0.02]` + `border-white/10`; accent cards: `bg-[var(--deck-accent)]/[0.03]` + `border-[var(--deck-accent)]/20`.

---

## Accessibility

- **Focus**: Skip link to first slide content; ensure dot buttons and arrow controls are focusable and have visible focus rings.
- **Landmarks**: Optional `aria-roledescription="slide"` on sections if you need screen-reader clarity (test with real SR).
- **`prefers-reduced-motion`**: Disable large parallax / stagger; keep `scrollIntoView` instant or minimal.
- **Contrast**: Check accent on dark bg (WCAG AA for interactive elements).

---

## Content mapping checklist

- [ ] Slide count matches source sections (1:1); title slide + N sections is fine.
- [ ] Each slide has mono section label `01 — Name` above the title.
- [ ] No slide combines unrelated ideas.
- [ ] Charts: labeled axes and sources; **no pie charts** (per skill).
- [ ] Comparison slide: ≤3–4 competitors, your row visually primary.

---

## Related skills (if installed)

- **`pitch-deck-visuals`** — slide framework, investor deck structure, data presentation.
- **`frontend-design`** / **`interface-design`** — distinctive marketing shell vs product UI patterns.

# Hackathon landing, SEO, PWA — reference

Main workflow: [SKILL.md](SKILL.md).

---

## Lenis + Framer Motion (App Router)

Install: `pnpm add lenis`. Import from `lenis/react`.

Framer integration (from upstream README): use `ReactLenis` with `options={{ autoRaf: false }}`, a `ref`, and `frame.update` / `cancelFrame` from `framer-motion` to drive `lenis.raf`. See [packages/react/README.md](https://github.com/darkroomengineering/lenis/blob/main/packages/react/README.md).

---

## Next.js metadata (conceptual)

- Place icons and `site.webmanifest` under **`public/`**.
- In `app/layout.tsx` (or route segment), export `metadata` with `icons` (multiple sizes), `manifest: '/site.webmanifest'`, `openGraph: { images: [{ url: '/site-banner.png', width: 1200, height: 630 }] }`, `twitter: { card: 'summary_large_image', images: ['/site-banner.png'] }`.
- Delete or replace **`app/favicon.ico`** so the default gray triangle does not override the branded set.

Adjust field names to match the **current** `Metadata` type in the project’s Next.js version.

---

## Landing page copywriting

### 1. WHO / WHY / WHAT

Headlines get far more attention than body copy. Structure:

- **Who** — audience
- **Why** — outcome they want
- **What** — your product’s role (one line)

**Example**

- **How engineers build better products**
- **Who:** engineers  
- **Why:** build better products  
- **What:** one platform to analyze, test, observe, and deploy features  

---

### 2. CTA → CTV

| Traditional CTA | Benefit-driven CTV |
|-----------------|-------------------|
| Do this | Here’s **why** you should |
| Get started | Try the app for **[concrete reason]** |
| Log in | Start **[user objective]** |

Lead with **why**, not only **what**.

---

### 3. PAS — Problem → Agitation → Solution

1. **Problem** — name the visitor’s situation.  
2. **Agitation** — make the cost of inaction clear.  
3. **Solution** — position the product as the relief.

**Sketch**

- **Problem:** small business owner struggling to get customers.  
- **Agitation:** empty pipeline → cash flow and stress.  
- **Solution:** marketing software that helps reach more of the right customers.  

---

### 4. Features tell, benefits sell

- **Features** — what it does (mechanism, specs).  
- **Benefits** — how life gets better (motivation, outcomes).

**Fitness tracker**

| Features | Benefits |
|----------|----------|
| Daily step count | Stay on track toward fitness goals |
| GPS health tracking | Context for activity without extra effort |

---

### 5. Social proof

- Reviews, testimonials, recognizable **logos**, awards/accreditations.  
- Short testimonial pattern: **Name (role)** + one specific outcome quote.

---

### 6. Visuals tell the story

Pair copy with imagery that carries **emotion** and **sequence** (problem → outcome). Example block:

- **Tagline:** For providers  
- **Heading:** Claims acceleration. Lightning-fast payment  
- **Supporting pillars:** error detection, predictive analytics, claim status, denial management  

---

## Fonts

### Body / UI (Google Fonts)

Prefer one primary sans for UI and long text: **Inter**, **Roboto**, **Questrial**, **Hind**, **Craftwork Sans**.  
**Helvetica** is system/UI-adjacent; on the web, pair with a licensed web alternative or system stack if you cannot embed it.

### Display / heading (stylized or bold)

Use for **hero and section titles** only; subset/limit weights for performance.

| Font | Notes | Link |
|------|--------|------|
| Safira March | Elegant, stylized | https://www.dafont.com/safira-march.font |
| Herkey | Elegant, stylized | https://www.dafont.com/herkey.font |
| Coolvetica | Bold sans | https://www.dafont.com/coolvetica.font |
| Relationship of Melodrame | Script, elegant | https://www.dafont.com/relationship-of-melodrame.font |
| The Magic Castle | Playful / game / script | https://www.dafont.com/the-magic-castle.font |

**Licensing:** DaFont assets are not all free for commercial use—confirm license before shipping a public hackathon demo or production.

**Implementation:** `next/font/google` for Google families; local `@font-face` (files in `public/fonts` or `app/fonts`) for display fonts with correct `font-display` (e.g. `swap`).

---

## Social image dimensions (verify before ship)

Platform requirements change. Before final export, confirm **Open Graph** and **Twitter/X** recommended dimensions in current docs; **1200×630** is a common OG default for `site-banner.png`.

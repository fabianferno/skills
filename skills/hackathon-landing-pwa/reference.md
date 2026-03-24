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

## Choosing fonts by app nature

Use this to **automatically** narrow choices before opening font sites. Mix **one Google body** + **one DaFont (or Google) display** unless the brand is intentionally single-family.

| App vibe / domain | Body (Google Fonts) | Heading / display (often DaFont) |
|-------------------|---------------------|----------------------------------|
| Default / unsure | **Inter** | **Coolvetica** |
| SaaS, dashboards, devtools | Inter, **Questrial**, Roboto | Coolvetica, **Herkey** (if a softer brand) |
| Fintech, trust, legal | Inter, **Hind**, Roboto | Coolvetica, Safira March (use sparingly) |
| Creative, portfolio, luxury | **Questrial**, Inter | **Safira March**, Relationship of Melodrame |
| Playful, gaming, consumer viral | Inter, Roboto | **The Magic Castle**, Coolvetica |
| Editorial, long-form reading | **Hind**, Inter | Herkey, Safira March |

When in doubt, ship **Inter + Coolvetica** and refine after user feedback.

---

## Fonts

### Body / UI (Google Fonts)

Prefer one primary sans for UI and long text: **Inter**, **Roboto**, **Questrial**, **Hind**, **Craftwork Sans**.  
**Helvetica** is system/UI-adjacent; on the web, pair with a licensed web alternative or system stack if you cannot embed it.

### Display / heading (DaFont or Google)

Use for **hero and section titles** only; subset/limit weights for performance. **Download** display fonts from [DaFont](https://www.dafont.com) (or the vendor site); verify **license** for demo vs commercial use.

| Font | Notes | Link |
|------|--------|------|
| Safira March | Elegant, stylized | https://www.dafont.com/safira-march.font |
| Herkey | Elegant, stylized | https://www.dafont.com/herkey.font |
| Coolvetica | Bold sans (default heading pair with Inter) | https://www.dafont.com/coolvetica.font |
| Relationship of Melodrame | Script, elegant | https://www.dafont.com/relationship-of-melodrame.font |
| The Magic Castle | Playful / game / script | https://www.dafont.com/the-magic-castle.font |

**Licensing:** DaFont listings vary—confirm redistribution and commercial use before ETHGlobal showcase or production.

---

## Tailwind font utilities

**Rule:** Every font gets a **kebab-case** Tailwind key: `font-coolvetica`, `font-safira-march`, etc., backed by a **`next/font` CSS variable** on the root layout. Apply **body** via `className` on `<html>` / `<body>`; apply **headings** with utilities on components (e.g. `className="font-coolvetica"`).

### Example: Inter (base) + Coolvetica (headings)

**1.** Add files: download Coolvetica, convert to **`.woff2`** if needed, e.g. `app/fonts/CoolveticaRg-Regular.woff2`.

**2.** Root layout (App Router):

```tsx
import { Inter } from 'next/font/google'
import localFont from 'next/font/local'

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
  display: 'swap',
})

const coolvetica = localFont({
  src: './fonts/CoolveticaRg-Regular.woff2',
  variable: '--font-coolvetica',
  display: 'swap',
})

// <html className={`${inter.variable} ${coolvetica.variable}`}>
//   <body className="font-sans antialiased">...</body>
// </html>
```

**3.** Tailwind v3 (`tailwind.config.ts`) — map `sans` to Inter and add **`font-coolvetica`**:

```ts
theme: {
  extend: {
    fontFamily: {
      sans: ['var(--font-inter)', 'ui-sans-serif', 'system-ui'],
      coolvetica: ['var(--font-coolvetica)', 'ui-sans-serif', 'system-ui'],
    },
  },
},
```

**4.** Usage — body inherits **`font-sans`**; headings use **`font-coolvetica`**:

```tsx
<h1 className="font-coolvetica text-4xl tracking-tight">Title</h1>
```

**Adding more DaFont or Google faces:** repeat `next/font/local` or `next/font/google` with `variable: '--font-{name}'`, add **`font-{kebab}`** under `theme.extend.fontFamily`, and compose variables on `<html className={...}>`.

**Tailwind v4:** define the same logical names inside `@theme` / `theme` per your project’s CSS-first config, still pointing at `var(--font-inter)`, `var(--font-coolvetica)`, etc.

---

## Social image dimensions (verify before ship)

Platform requirements change. Before final export, confirm **Open Graph** and **Twitter/X** recommended dimensions in current docs; **1200×630** is a common OG default for `site-banner.png`.

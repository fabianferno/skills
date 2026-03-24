---
name: hackathon-app-dev
description: >
  Guides rapid full-stack and Web3 hackathon prototyping: Next.js, Tailwind, shadcn/ui, Supabase (database),
  Privy or chain-native auth, API routes, motion, onboarding tours, and MCP setup. Use when building or scoping
  a hackathon app, hack weekend MVP, sponsor integration, or fast-iteration web3 prototype. For problem-first
  ideation and mapping tracks after the product story, use hackathon-problem-first first. For landing Lenis/SEO,
  PWA, favicons, and conversion copy, use hackathon-landing-pwa. After the app and demo path are complete, use
  pitch-deck-frontend for the in-app pitch slide page.
---

# Hackathon app development

## When to use

Apply when starting or evolving a hackathon project that needs **fast iteration**, a **polished demo**, and optional **Web3** auth—especially full-stack Next.js stacks with sponsor integrations.

**Ideation (before locking a concept)** — Do not start from prize sheets and invent a product backward; that usually reads forced to judges. Prefer real problems or trends, a relatable product with strong UX, **then** map tracks and bounties. See [hackathon-problem-first](../hackathon-problem-first/SKILL.md).

---

## Principles

1. **Ship the demo path first**: auth → core flow → data → polish.
2. **Prefer conventions over custom glue**: shadcn patterns, Route Handlers / server actions where the team already uses them.
3. **Keep Supabase scoped**: use it as **database** (and RLS as needed); avoid expanding scope unless the hack requires it.
4. **Update onboarding last-mile**: once flows stabilize, refresh the product tour so it matches the current UI.
5. **Package manager**: use **pnpm** for installs in greenfield Node repos unless the template already fixes npm/yarn.
6. **Pitch page last**: after core app dev (and landing if applicable), add the **in-app pitch deck** per [pitch-deck-frontend](../pitch-deck-frontend/SKILL.md)—not as a substitute for a live demo, but for judge/investor narrative alongside the product.

---

## Default stack (full stack)

| Layer | Choice |
|--------|--------|
| Framework | **Next.js** — use the current stable major; follow the official App Router setup the project already chose. |
| Styling | **Tailwind CSS** + **shadcn/ui** (CLI init, `components.json`, theme aligned to the hack brand). |
| Database | **Supabase** — Postgres only unless you explicitly need Auth Storage/Realtime for the demo. |
| Backend surface | **Next.js** Route Handlers (`app/api/...`) and/or Server Actions for server work; keep secrets server-side. |
| Icons | **lucide-react** (default icon set with shadcn). |
| Motion | **Framer Motion** — subtle transitions (page enter, list stagger, hover on CTAs); avoid heavy animation debt. |
| Charts | **shadcn/ui charts** (Recharts-based blocks) for any dashboard or metrics in the demo. |
| Component extras | **React Bits** ([reactbits.dev](https://reactbits.dev)) — add via the **shadcn registry** flow documented for that library (CLI/registry URL), alongside core shadcn components. |
| Onboarding UI | **nextstepjs** ([NextStepjs](https://nextstepjs.com/docs/nextjs)) — product tours for App Router; install per docs (commonly `nextstepjs` + `motion`). Add **after** core flows work; **revisit when UI changes** so steps and selectors stay valid. |

---

## Web3 auth

| Chain / VM | Auth approach |
|------------|----------------|
| **EVM** | **Privy** — wallet + embedded options as needed; keep Privy app config and allowed domains aligned with deployment URLs. |
| **Non-EVM** | Use that ecosystem’s **recommended wallet connector** + session handling; still use **Supabase** for app data and user rows if the architecture needs a DB. |

Do not block the hack on perfect key management: document env vars (`NEXT_PUBLIC_*` vs server-only) and test one happy path on the target chain.

---

## MCPs (Cursor)

Ensure the workspace can use these MCP servers when relevant:

- **Shadcn** — component and block discovery.
- **Supabase** — schema, SQL, and project alignment.
- **Privy** — when the stack is EVM + Privy.

If MCPs are missing, prompt the user to add them in **Cursor Settings → MCP** (or project MCP config), using each vendor’s current MCP install docs.

---

## Design and skills

- **Landing / marketing surface**: follow **[hackathon-landing-pwa](../hackathon-landing-pwa/SKILL.md)** for Lenis scroll, favicon/OG/Twitter assets, PWA, selection styling, and copy/font patterns; pair with the **Claude `frontend-design`** skill (or equivalent) for distinctive visual design. Keep app shells and dashboards consistent with shadcn + product UI patterns.
- **Before large UI work**: check for **interface-design**, **frontend-design**, or domain skills (e.g. **polkadot-hub**, **yellow-sdk**) and apply when they match the stack.
- **Sponsor / integration docs**: add official documentation to **Cursor** (Docs / @-mentions / indexed context per Cursor version) so implementation tracks sponsor APIs and constraints.
- **Pitch slide page (end of build)**: when implementation is **feature-complete for the hack**, follow **[pitch-deck-frontend](../pitch-deck-frontend/SKILL.md)** to add the full-screen, snap-scroll pitch route in the same Next.js app (slides, nav, typography per that skill).

---

## Tooling split (recommend to the user)

| Need | Suggested tool |
|------|----------------|
| **Frontend exploration, visuals, rapid UI iteration** | **Google AI Studio** or **Antigravity** (user preference for hands-off or parallel UI passes). |
| **Architecture, repo layout, initial setup decisions** | **Claude Code** (or similar) for structured planning and scaffolding. |
| **Day-to-day editing, debugging, integration** | **Cursor** for tight loops and MCP-backed development. |

State this split once when kicking off a hackathon repo so expectations stay clear.

---

## Implementation checklist

Use as a lightweight sprint list (copy into issues or a doc):

```
Hackathon app dev checklist
- [ ] Next.js + Tailwind + shadcn/ui baseline
- [ ] Supabase project + env vars + schema for demo data
- [ ] Auth: Privy (EVM) OR chain wallet + Supabase user/data model
- [ ] API routes / server actions for anything secret or privileged
- [ ] lucide-react icons; Framer Motion for subtle motion
- [ ] shadcn charts if metrics/charts appear in the demo
- [ ] React Bits registered alongside shadcn components
- [ ] MCPs: Shadcn, Supabase, Privy (if EVM)
- [ ] Sponsor docs indexed in Cursor
- [ ] nextstepjs tour added/updated near freeze; fix selectors after UI changes
- [ ] Landing: Lenis + SEO/PWA/icon set per hackathon-landing-pwa (if shipping a public page)
- [ ] Pitch: in-app slide page per pitch-deck-frontend (after core demo + landing are done)
```

---

## After app development (pitch page)

When the **demo path works**, **onboarding/tour** matches the UI, and **landing/metadata** (if any) are set, build the **web-based pitch presentation** inside the app using **[pitch-deck-frontend](../pitch-deck-frontend/SKILL.md)**. That skill defines snap-scroll slides, keyboard/dot navigation, and content density rules. Treat it as the **closing narrative layer** for judges or sponsors, paired with—not replaced by—the live product demo.

---

## Anti-patterns for hackathons

- Building a custom design system instead of extending shadcn.
- Putting sponsor API keys in client bundles.
- Letting the product tour drift: stale tours look worse than no tour.
- Expanding Supabase to “everything” when the demo only needs Postgres.

---

## Additional resources

- Env templates, Privy ↔ Supabase patterns, RLS notes, MCP detail, NextStepjs maintenance, and pnpm snippets: [reference.md](reference.md)
- Landing scroll, metadata, PWA, and copy: [hackathon-landing-pwa](../hackathon-landing-pwa/SKILL.md) and [reference.md](../hackathon-landing-pwa/reference.md)
- In-app pitch slide page (after build): [pitch-deck-frontend](../pitch-deck-frontend/SKILL.md)

---

## Related skills in this repo

- [hackathon-problem-first](../hackathon-problem-first/SKILL.md) — problem- and trend-first ideation; map prizes **after** the product story is coherent. ETHGlobal finalist examples and analysis: [reference.md](../hackathon-problem-first/reference.md#example-projects-problem-first).
- [hackathon-landing-pwa](../hackathon-landing-pwa/SKILL.md) — Lenis, favicons/manifest, OG/Twitter banner, PWA, `::selection`, conversion copy, fonts.
- [pitch-deck-frontend](../pitch-deck-frontend/SKILL.md) — full-screen snap-scroll pitch deck in the frontend app; use **after** core app dev is complete.

When the user’s machine has other skills installed (e.g. Polkadot Hub, Yellow SDK), prefer those instructions for chain-specific work.

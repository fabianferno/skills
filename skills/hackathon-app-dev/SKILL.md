---
name: hackathon-app-dev
description: >
  Guides rapid full-stack and Web3 hackathon prototyping: Next.js, Tailwind, shadcn/ui, Supabase (database),
  Privy or chain-native auth, API routes, motion, onboarding tours, and MCP setup. Use when building or scoping
  a hackathon app, hack weekend MVP, sponsor integration, or fast-iteration web3 prototype. For problem-first
  ideation and mapping tracks after the product story, use hackathon-problem-first first.
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

- **Landing / marketing surface only**: prefer the **Claude `frontend-design`** skill (or equivalent in the user’s environment) for a distinctive first impression; keep app shells and dashboards consistent with shadcn + product UI patterns.
- **Before large UI work**: check for **interface-design**, **frontend-design**, or domain skills (e.g. **polkadot-hub**, **yellow-sdk**) and apply when they match the stack.
- **Sponsor / integration docs**: add official documentation to **Cursor** (Docs / @-mentions / indexed context per Cursor version) so implementation tracks sponsor APIs and constraints.

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
```

---

## Anti-patterns for hackathons

- Building a custom design system instead of extending shadcn.
- Putting sponsor API keys in client bundles.
- Letting the product tour drift: stale tours look worse than no tour.
- Expanding Supabase to “everything” when the demo only needs Postgres.

---

## Additional resources

- Env templates, Privy ↔ Supabase patterns, RLS notes, MCP detail, NextStepjs maintenance, and pnpm snippets: [reference.md](reference.md)

---

## Related skills in this repo

- [hackathon-problem-first](../hackathon-problem-first/SKILL.md) — problem- and trend-first ideation; map prizes **after** the product story is coherent. ETHGlobal finalist examples and analysis: [reference.md](../hackathon-problem-first/reference.md#example-projects-problem-first).
- `hackathon-idea-generator` — many candidate ideas and track constraints (if installed).
- `hackathon-judge-simulator` — pitch and demo hardening (if installed).
- `interface-design` — product UI patterns, non-marketing (if installed).

When the user’s machine has other skills installed (e.g. Polkadot Hub, Yellow SDK), prefer those instructions for chain-specific work.

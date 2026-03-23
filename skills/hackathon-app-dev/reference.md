# Hackathon app dev — reference

Deep detail for `hackathon-app-dev`. Read when implementing auth, env, database, or Cursor MCP setup.

---

## Environment variables

### Naming split (Next.js)

| Exposure | Pattern | Examples |
|----------|---------|----------|
| Browser-safe | `NEXT_PUBLIC_*` | Supabase anon key, Privy app id, chain RPC URL if intentionally public |
| Server-only | no `NEXT_PUBLIC_` | Supabase service role, sponsor API keys, webhook secrets, Privy app secret |

**Rule**: If it can mint money, read private data, or impersonate users, it stays on the server (Route Handler, Server Action, or server component without leaking to client props).

### Starter `.env.local` template (adjust names to your stack)

```bash
# Supabase (public + server)
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Privy (EVM)
NEXT_PUBLIC_PRIVY_APP_ID=
PRIVY_APP_SECRET=

# Sponsor / integrations (server-only unless docs say otherwise)
# SPONSOR_API_KEY=
```

Never commit `.env.local`. Document required keys in `README.md` for teammates.

---

## Privy (EVM) + Supabase (DB only)

Typical hackathon shape:

1. User signs in with **Privy** on the client.
2. App obtains a **stable user identifier** from Privy (`user.id` or linked wallet address — follow Privy’s current API).
3. On first login, **upsert** a row in Supabase (e.g. `profiles` or `users`) keyed by that id or by `wallet_address`.
4. All app-specific data references that row’s primary key.

**Server boundary**: Prefer a Route Handler or Server Action that:

- Verifies the Privy session (or JWT) server-side per Privy docs.
- Then runs Supabase queries with **`SUPABASE_SERVICE_ROLE_KEY`** only on the server for admin paths, **or** uses the user-scoped client with **RLS** policies tied to a claim you set (see below).

**Anti-pattern**: Trusting `wallet` or `userId` from the client body without verification.

---

## Non-EVM wallet + Supabase

1. Connect with the chain’s standard wallet kit.
2. Produce a **session** or **signed message** flow if you need “logged in” state.
3. Store the same mapping: `chain_account_id` / `public_key` → Supabase profile row.

Keep verification and any “who is this user?” logic on the server when possible.

---

## Supabase RLS (minimum viable for demos)

If the client reads/writes with the **anon key**, add policies so users only touch their rows:

- Table holds `user_id` (or `wallet_address`) matching your auth identity.
- Policy examples (conceptual): `select`/`insert`/`update` where `user_id = current_setting` or where JWT claim matches — exact SQL depends on whether you use Supabase Auth or only custom JWT.

For a **very short** internal-only demo, some teams use the service role only from Next.js APIs and skip client-side Supabase writes; that trades speed for less “real” client security—call it out in README if you do it.

---

## Next.js backend surface

| Use case | Pattern |
|----------|---------|
| Secret keys, sponsor APIs | `app/api/.../route.ts` or Server Action with `'use server'`; no secrets in client bundles |
| Reads that depend on cookies / session | Server Component or Route Handler |
| File uploads | Route Handler + size limits; validate MIME; scan sponsor limits |

Add **rate limiting** only if the sponsor API or abuse risk requires it in the timeframe.

---

## Cursor MCPs

| MCP | Typical use |
|-----|-------------|
| Shadcn | Search/add components and blocks without leaving editor |
| Supabase | Inspect schema, draft SQL/migrations, align types |
| Privy | Auth snippets and config alignment with current docs |

Configure via **Cursor Settings → MCP** or the project’s MCP config file (path varies by Cursor version). Use each vendor’s official MCP installation instructions; URLs change.

---

## NextStepjs (nextstepjs) maintenance

- Prefer **stable `data-*` attributes or test ids** on tour targets instead of brittle CSS paths.
- After changing layout or shadcn component structure, **re-run the tour** once locally.
- If using cross-route steps, confirm `nextRoute` / `prevRoute` (per [NextStepjs routing docs](https://nextstepjs.com/docs/nextjs/routing)) still match the App Router tree.

---

## Sponsor docs in Cursor

1. Add official docs as **indexed documentation** or **@Docs** sources (whichever your Cursor build supports).
2. Include **API reference + auth** pages for each sponsor integration.
3. For one-off PDFs or Notion pages, paste critical endpoints/limits into a repo `docs/sponsors.md` and reference that file in chat.

---

## pnpm commands (quick copy)

```bash
pnpm create next-app@latest
pnpm dlx shadcn@latest init
pnpm add lucide-react framer-motion nextstepjs motion
# Supabase client (if using from Next.js)
pnpm add @supabase/supabase-js
# Privy (EVM)
pnpm add @privy-io/react-auth
```

Exact Privy / wallet packages depend on the chosen provider version—confirm on their install pages before locking versions for the weekend.

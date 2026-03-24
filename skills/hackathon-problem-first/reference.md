# Hackathon problem-first — reference

Workshop-style prompts and lightweight artifacts to use **during** ideation. Philosophy and order of operations: [SKILL.md](SKILL.md).

---

## Example projects (problem-first)

These ETHGlobal showcase builds won **finalist** placement and **multiple prizes**. They are useful as **counterexamples to prize-first hacks**: you could name-check the same sponsors, but the **demo narrative** starts from a situation judges and users already understand.

**Pattern to copy**: state the **anchor problem in plain language** → show **one core loop** → only then explain which integrations make that loop trustworthy, social, or automatable.

| Project | Showcase | Problem anchor (no sponsor names) | How sponsor / chain tech **serves** the product (not the reverse) |
|---------|----------|-----------------------------------|---------------------------------------------------------------------|
| **Wrld Map** | https://ethglobal.com/showcase/wrld-map-v64h2 | Travel confirmations live in email; there is no single, **verifiable**, shareable map of where you have really been. | **Email proofs** turn receipts into facts; **World ID** ties maps to real humans; **on-chain storage** + explorer (e.g. Blockscout) make the history portable and inspectable—**the product is the atlas**, not “a zk demo.” |
| **CalCast** | https://ethglobal.com/showcase/calcast-7g042 | On Farcaster you discover people, then **leave** to Calendly-style tools to book time—extra apps, weaker ownership of scheduling data. | **Frames** keep booking **inside** the feed; **indexing** (e.g. The Graph) and **chain** deployment support the same loop a user would describe as “book a call where I already am.” |
| **CalendeFi** | https://ethglobal.com/showcase/calendefi-6d3ji | DeFi forces new UIs and habits; **millions of people already live in calendars** for “when things happen,” including money and coordination. | Calendar events become **intents**; **WalletConnect**, DEX, fiat rails, **ENS** are **execution and identity layers** for scheduling you already understand—**the product is calendar-native finance**, not “another wallet tab.” |
| **WalletSheets** | https://ethglobal.com/showcase/walletsheets-g5p8i | Crypto power users still fight specialized dashboards; **spreadsheets** are a global default for tracking and “what-if” work. | Agents, **private compute / vaults**, and **WalletConnect** let a Sheet **act** as a wallet and DeFi surface—**the product is Sheets as command center**, not “Nillion for its own sake.” |
| **OnlyCars** | https://ethglobal.com/showcase/onlycars-vo161 | EV drivers need to **find**, **pay for**, and **share** charging without brittle siloed apps; coordination and trust matter at the edge. | Identity, messaging, indexing, and contracts **support find → pay → share charger network**—**the product is charging that feels like one app**, not a stack parade. |
| **Atestamint** | https://ethglobal.com/showcase/atestamint-xi8ch | NFT **rug pulls** destroyed trust; backers have little **on-chain** assurance that creators will ship milestones before funds fully move. | **Vaults**, **attestations**, **sybil-resistant humans**, **NFT issuance**, and **indexing** implement **milestone unlocks and reputation**—**the product is provable delivery**, not “use EAS + Zora.” |

**Takeaway for your pitch**: if you remove sponsor logos and the story collapses, you are still prize-first. If the story stands and integrations are “how we made it real,” you are aligned with this method.

---

## One-page order (print or duplicate in a doc)

1. **Anchor** — Real pain or credible trend (no sponsor names).
2. **User + job-to-be-done** — Who acts, in what situation?
3. **MVP loop** — One core flow that works without a hackathon.
4. **UX bar** — What must feel obvious to a non-expert in the demo?
5. **Prize fit** — Primary track + honest secondary tags (optional).

---

## Anchor prompts (pick one and complete)

| Prompt | Your one-liner |
|--------|----------------|
| “Every week, X wastes time/money on ___ because ___.” | |
| “When Y happens, people currently use Z and hate it because ___.” | |
| “A trend that is obviously accelerating is ___; that breaks ___ for ___.” | |

**Gate**: If you cannot explain the anchor **without** naming a sponsor API, sharpen the problem first.

---

## Product definition template

```text
User: [role, context]
Trigger: [when they reach for a solution]
Core action: [single verb — e.g. reconcile, prove, schedule, settle]
Outcome: [measurable or visceral win in < 2 minutes of demo]
Out of scope for weekend: [explicit cuts]
```

---

## Prize mapping matrix (after the story is solid)

| Track / bounty | Natural fit? (Y / stretch / no) | Evidence from product (one sentence) |
|----------------|----------------------------------|--------------------------------------|
| | | |
| | | |

**Rule**: Exactly **one** primary row should read “this is the spine of the demo.” Everything else is supporting evidence, not a rewrite of the pitch.

---

## Pitch outline (judge-facing)

1. **Problem** — Anchor + who hurts (10–20s).
2. **Insight** — Why now / why existing tools fail (10–15s).
3. **Product** — What you built + core loop (30–45s).
4. **Demo** — Live path, one happy case (60–90s).
5. **Traction / ask** — Optional; only if true (10–20s).
6. **Prize** — One sentence tying **primary** alignment (5–10s).

---

## Red flags (stop and fix)

- The **first** slide mentions three sponsors and zero users.
- Demo is “we called API A then API B” with no recognizable scenario.
- You need **five** prize rows marked “primary.”
- Replacing the problem statement changes **every** screen in the mock — prize was driving design.

---

## Handoff

- Implementation stack and checklist: **[hackathon-app-dev](../hackathon-app-dev/SKILL.md)** (+ [reference.md](../hackathon-app-dev/reference.md)). The case studies above are **examples of that build quality** once the concept is fixed.
- Many ideas / track constraints: **`hackathon-idea-generator`** (if installed).
- Adversarial Q&A: **`hackathon-judge-simulator`** (if installed).

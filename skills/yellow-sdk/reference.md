# Yellow SDK — reference

Condensed lookup while wiring **Nitrolite (ERC-7824)** over WebSocket. For full flows and code samples, see [SKILL.md](SKILL.md).

---

## Endpoints

| Environment | WebSocket URL |
|-------------|----------------|
| Sandbox (dev) | `wss://clearnet-sandbox.yellow.com/ws` |
| Production | `wss://clearnet.yellow.com/ws` |

**Dynamic config**: Call **`get_config`** (public, no signature) for current chains, custody/adjudicator addresses, and broker address. Prefer this over hardcoding for anything beyond quick local tests.

---

## Environment variables

| Variable | Role |
|----------|------|
| `PRIVATE_KEY` | Dev wallet (never commit) |
| `ALCHEMY_RPC_URL` / `SEPOLIA_RPC_URL` / `BASE_RPC_URL` | Chain RPC |
| `CLEARNODE_WS_URL` | Optional override for WS endpoint |

Use **pnpm** in greenfield Node repos unless the project already standardizes on another manager.

---

## Packages (pnpm)

```bash
pnpm add @erc7824/nitrolite viem dotenv
pnpm add -D typescript @types/node tsx
```

Set `"type": "module"` and `tsconfig` **`moduleResolution`**: `"bundler"` or `"node16"` for viem compatibility.

---

## Wire protocol shape

- Requests/responses: JSON-RPC–style over WebSocket.
- Successful handler path: **`response.res[1]`** = method name (snake_case), **`response.res[2]`** = params (snake_case keys: `channel_id`, `state_data`, `server_signature`, etc.).
- Errors: method **`error`** with descriptive string (not numeric codes).

### Methods to handle in `ws.on('message')`

| `res[1]` | Typical next step |
|----------|-------------------|
| `auth_challenge` | EIP-712 sign with **main wallet**, send `auth_verify` |
| `auth_verify` | JWT / session ready; subsequent RPC signed with **session** signer |
| `get_config` | Cache chains, contracts, `brokerAddress` |
| `create_channel` | Submit `NitroliteClient.createChannel` with server signature |
| `resize_channel` | Map state, then `nitroliteClient.resizeChannel` |
| `close_channel` | Then `closeChannel`, then `withdrawal` if needed |
| `channels` | List open channels (useful for cleanup) |
| `get_ledger_balances` | Unified balance read |
| `transfer` | Unified ledger transfer result |
| `error` | Log / surface `params` message |

---

## `resize_amount` vs `allocate_amount`

| Field | Meaning |
|-------|---------|
| **`resize_amount`** | L1 custody → channel. Requires prior **on-chain deposit** to custody; otherwise `InsufficientBalance`. |
| **`allocate_amount`** | **Unified balance** (off-chain ledger, e.g. Sandbox faucet) → channel. Use for sandbox faucet flows. |

**`funds_destination`**: For allocation from unified balance, use the **wallet address**. For moving back to the unified ledger, use the **broker** address from `get_config`.

---

## Session keys (quick rules)

- One session key per wallet + application; new registration invalidates the previous.
- Set **allowances** and **short `expires_at`** (e.g. 1–24h). Empty allowances = no spending.
- **`auth_verify`** must be signed by the **main wallet** (EIP-712), not the session key.
- Challenge ~**5 minutes**; on expiry restart from `auth_request`.
- v0.5.0+: for **new** channels, on-chain participant is the **wallet address**; session key still signs RPC.

---

## Sandbox faucet (unified balance)

```bash
curl -X POST https://clearnet-sandbox.yellow.com/faucet/requestTokens \
  -H "Content-Type: application/json" \
  -d '{"userAddress":"<your_wallet_address>"}'
```

---

## Reference contract addresses (verify with `get_config`)

**Base (mainnet)** — custody `0x490fb189DdE3a01B00be9BA5F41e3447FbC838b6`, adjudicator `0x7de4A0736Cf5740fD3Ca2F2e9cc85c9AC223eF0C`, USDC `0x833589fcd6edb6e08f4c7c32d4f71b54bda02913`.

**Sepolia (sandbox)** — custody `0x019B65A265EB3363822f2752141b3dF16131b262`, adjudicator `0x7c7ccbc98469190849BCC6c926307794fDfB11F2`, ytest.usd `0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238`.

---

## Troubleshooting (symptom → likely fix)

| Symptom | Check |
|---------|--------|
| `InsufficientBalance` on resize | Using `resize_amount` without custody deposit → use **`allocate_amount`** from unified balance |
| `Invalid signature` on auth | `auth_verify` must be **main wallet** EIP-712 |
| Challenge / session errors | Challenge TTL ~5 min; expired session → full re-auth |
| `operation denied: insufficient session key allowance` | New session with higher allowance or use root signer |
| `non-zero allocation` / funding timeouts | Close stale channels (`channels` + `close_channel` + on-chain close) |

---

## Official docs

- [Yellow docs](https://docs.yellow.org/)
- [What is Yellow SDK](https://docs.yellow.org/docs/learn/introduction/what-is-yellow-sdk)
- [Authentication](https://docs.yellow.org/docs/protocol/off-chain/authentication)
- [ERC-7824](https://erc7824.org/)

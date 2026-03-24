---
name: yellow-sdk
description: Implement Yellow SDK (WebSocket gateway for Nitrolite/ERC-7824 state channels) for instant off-chain transactions, payment channels, and multi-party app sessions. Use when working with Yellow Network, Nitrolite protocol, state channels, payment channels, off-chain transactions, or ERC-7824.
---

# Yellow SDK

Yellow SDK is a real-time communication toolkit built on the **Nitrolite** protocol (ERC-7824). It has two main components:

- **Nitrolite RPC**: API for quick integration with the Yellow Network (WebSocket, signed JSON-RPC–style messages).
- **NitroliteClient**: For low-level control over state channels (create, resize, close, deposit, withdrawal on-chain).

Yellow Network is a decentralized clearing and settlement network: cross-chain liquidity, high-frequency transfers without gas, peer-to-peer asset transfers, and chain abstraction via **unified balance** across multiple chains. Users connect through **Clearnodes** (trustless execution layers) that can process very high off-chain throughput.

## Prerequisites & Environment

- **Node.js**: 18.x minimum, 20.x recommended.
- **Package manager**: npm, yarn, or pnpm (use pnpm when specified by project).
- **Core dependencies**: `@erc7824/nitrolite`, `viem`. Dev: `typescript`, `@types/node`, `tsx`, `dotenv`.
- **TypeScript**: Use `"moduleResolution": "bundler"` or `"node16"`; set `"type": "module"` in `package.json` for ESM.
- **Environment variables** (e.g. `.env`, never commit):
  - `PRIVATE_KEY` — wallet private key (dev only).
  - `ALCHEMY_RPC_URL` or `SEPOLIA_RPC_URL` / `BASE_RPC_URL` — RPC for the chain(s) you use.
  - `CLEARNODE_WS_URL` — WebSocket URL (see Endpoints below).

**Recommended project structure:**

```
yellow-app/
├── src/
│   ├── index.ts       # Entry point
│   ├── config.ts      # Configuration
│   ├── client.ts      # Nitrolite client setup
│   ├── auth.ts        # Authentication logic
│   └── channels/
│       ├── create.ts  # Channel creation
│       ├── transfer.ts
│       └── close.ts
├── scripts/
│   └── create-wallet.ts
├── .env
├── package.json
└── tsconfig.json
```

## Endpoints

- **Production**: `wss://clearnet.yellow.com/ws`
- **Sandbox** (testing): `wss://clearnet-sandbox.yellow.com/ws`

Use Sandbox for development. Contract addresses and supported chains are **dynamic**: query `get_config` (public, no signature) to get current chains and contract addresses.

## Core Concepts

### Yellow Network & Clearnode
- WebSocket gateway to the Nitrolite protocol; real-time communication, reconnection, message queuing.
- **Clearnode**: Off-chain service that manages Nitro RPC, provides **unified balance**, coordinates channels, hosts app sessions. Centralized for speed but trustless via ERC-7824 (challenge/dispute and on-chain settlement).

### Nitrolite Protocol (ERC-7824)
- On-chain state channel protocol: instant, low-cost off-chain updates with on-chain settlement.
- **Nitro RPC**: Off-chain protocol (e.g. NitroRPC/0.4); compact JSON, every message signed, bidirectional.
- **Unified Balance**: Aggregated funds across all supported chains (e.g. deposit on Polygon, withdraw on Base).

### Key Components
- **Session Keys**: Temporary ECDSA keypairs for signing RPC messages without main wallet; allowances and expiration apply.
- **Channels**: Off-chain payment channels between participants (user + Clearnode); on-chain representation has participants, adjudicator, challenge duration, nonce → `channelId = keccak256(...)`.
- **App Sessions**: Multi-party state channels on top of unified balance; quorum/weights for governance.
- **Custody Contract**: On-chain contract holding deposited funds; implements create, close, resize, deposit, withdrawal.
- **Adjudicator**: Defines rules for valid state transitions (e.g. SimpleConsensus, Remittance).

## Key Terms & Protocol Details

- **State**: Snapshot of channel (intent, version, data, allocations, sigs). Higher version supersedes lower. Intents: `INITIALIZE` (create), `OPERATE` (off-chain updates), `RESIZE`, `FINALIZE` (close).
- **Allocation**: `{ destination, token, amount }`; sum of allocations = total funds in channel.
- **Channel states**: VOID → INITIAL → ACTIVE → (DISPUTE) → FINAL.
- **Challenge period**: Time window for dispute response (e.g. 3600–86400 seconds); typical default 24h.
- **Nitro RPC message format**: `[requestId, method, params, timestamp]`. Responses: `res` array where `res[1]` = method name (snake_case), `res[2]` = params (snake_case: e.g. `channel_id`, `state_data`, `server_signature`). Wire method names: `auth_challenge`, `auth_verify`, `create_channel`, `resize_channel`, `close_channel`, `get_config`, `channels`, `get_ledger_balances`, `transfer`, `error`.
- **get_config**: Public endpoint; no signature. Returns supported chains, contract addresses, broker address. Use for dynamic config.
- **Session keys (v0.5.0+)**: For **new** channels, the **wallet address** is used as the on-chain participant, not the session key. Session keys still used for RPC signing and for backward compatibility with channels created before v0.5.0.

## Authentication Flow

Authentication can be **main wallet (root signer)** for every request, or **session keys** (3-step challenge-response). Session keys: one-time EIP-712 sign with main wallet, then sign RPC with session key; optional allowances and expiration.

- **auth_request**: Public; no signature. Send address, session_key, application, allowances, scope, expires_at. Server responds with `auth_challenge` containing `challenge_message` (UUID; single-use, expires in 5 minutes).
- **auth_verify**: Sign **with main wallet** (EIP-712) the challenge + scope, wallet, session_key, expires_at, allowances. Or send existing `jwt` to re-authenticate without signing. Response includes `jwt_token`, `success`.
- When re-authenticating with an **already registered** session key, application/allowances/scope/expires_at in the request are ignored (initial registration is used).

### Step 1: Setup Wallet and Session Key

```typescript
import { createWalletClient, createPublicClient, http } from 'viem';
import { privateKeyToAccount, generatePrivateKey } from 'viem/accounts';
import { base } from 'viem/chains';
import { 
    createAuthRequestMessage, 
    createAuthVerifyMessageFromChallenge,
    createEIP712AuthMessageSigner,
    createECDSAMessageSigner,
    RPCMethod,
    RPCResponse
} from '@erc7824/nitrolite';

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);
const walletClient = createWalletClient({ account, chain: base, transport: http() });

const sessionPrivateKey = generatePrivateKey();
const sessionAccount = privateKeyToAccount(sessionPrivateKey);
const sessionSigner = createECDSAMessageSigner(sessionPrivateKey);
```

### Step 2: Connect to Yellow Network

Use raw WebSocket or a wrapper (e.g. `yellow-ts` Client). Example with WebSocket:

```typescript
import WebSocket from 'ws';

const ws = new WebSocket('wss://clearnet-sandbox.yellow.com/ws'); // or clearnet.yellow.com/ws for production
```

### Step 3: Request Authentication

```typescript
const authParams = {
    session_key: sessionAccount.address,
    allowances: [{ asset: 'usdc', amount: '1000000000' }],
    expires_at: BigInt(Math.floor(Date.now() / 1000) + 3600), // 1 hour, seconds
    scope: 'your.app.scope',
};

const authMessage = await createAuthRequestMessage({
    address: account.address,
    application: 'Your App Name',
    ...authParams,
});

ws.send(authMessage);
```

### Step 4: Handle auth_challenge and auth_verify

Response format: `response.res[1]` = method (e.g. `'auth_challenge'`), `response.res[2]` = params. Use `createAuthVerifyMessageFromChallenge(signer, challenge)` with the challenge from the server:

```typescript
ws.on('message', async (data) => {
    const response = JSON.parse(data.toString());
    if (response.error) { /* handle */ return; }
    const method = response.res?.[1];
    const params = response.res?.[2];

    if (method === 'auth_challenge') {
        const challenge = params.challenge_message;
        const signer = createEIP712AuthMessageSigner(walletClient, authParams, { name: 'Your App Name' });
        const verifyMsg = await createAuthVerifyMessageFromChallenge(signer, challenge);
        ws.send(verifyMsg);
    } else if (method === 'auth_verify' && params?.success) {
        console.log('✅ Authenticated');
        // Subsequent requests signed with sessionSigner
    } else if (method === 'error') {
        console.error('❌ Error:', params);
    }
});
```

## Common Operations

### Get config (supported chains and contracts)

`get_config` is public (no signature). Use it to get current custody/adjudicator addresses and broker address for resize/allocate:

```typescript
import { createGetConfigMessage } from '@erc7824/nitrolite';

const configMsg = await createGetConfigMessage(sessionSigner);
ws.send(configMsg);
// On response.res[1] === 'get_config', use response.res[2].chains, .contracts, .brokerAddress
```

### Create Payment Channel

Server responds with `create_channel`; params use snake_case: `channel_id`, `channel`, `state` (with `state_data`, `allocations`), `server_signature`. Submit to chain with `NitroliteClient.createChannel`.

```typescript
import { NitroliteClient, WalletStateSigner, createCreateChannelMessage } from '@erc7824/nitrolite';

// Contract addresses: get from get_config or use known values (see Contract Addresses below)
const createChannelMessage = await createCreateChannelMessage(sessionSigner, {
    chain_id: base.id,
    token: USDC_TOKEN_BASE as `0x${string}`,
});

ws.send(createChannelMessage);

// On response.res[1] === 'create_channel':
const { channel_id, channel, state, server_signature } = response.res[2];
const unsignedInitialState = {
    intent: state.intent,
    version: BigInt(state.version),
    data: state.state_data,
    allocations: state.allocations.map((a: any) => ({
        destination: a.destination,
        token: a.token,
        amount: BigInt(a.amount),
    })),
};
const { channelId, txHash } = await nitroliteClient.createChannel({
    channel,
    unsignedInitialState,
    serverSignature: server_signature,
});
```

### Deposit to Custody

```typescript
import { NitroliteClient, WalletStateSigner } from '@erc7824/nitrolite';
import { parseUnits } from 'viem';

const nitroliteClient = new NitroliteClient({
    walletClient,
    publicClient,
    stateSigner: new WalletStateSigner(walletClient),
    addresses: {
        custody: '0x490fb189DdE3a01B00be9BA5F41e3447FbC838b6',
        adjudicator: '0x7de4A0736Cf5740fD3Ca2F2e9cc85c9AC223eF0C',
    },
    chainId: base.id,
    challengeDuration: 3600n,
});

const depositAmount = parseUnits('1.0', 6); // USDC has 6 decimals
const txHash = await nitroliteClient.deposit(USDC_TOKEN_BASE, depositAmount);
console.log(`Deposit tx: ${txHash}`);
```

### Resize Channel: resize_amount vs allocate_amount

- **resize_amount**: Moves funds from **L1 Custody** (on-chain deposit) into the channel. Only use if you have already deposited to the custody contract; otherwise you get `InsufficientBalance`.
- **allocate_amount**: Moves funds from your **Unified Balance** (off-chain, e.g. Sandbox faucet) into the channel. For Sandbox testing with faucet tokens, use **allocate_amount**, not resize_amount.
- **funds_destination**: For **allocation** (unified balance → channel) use your **wallet address**. For moving from channel back to unified ledger use **broker** address (from `get_config`).

```typescript
import { createResizeChannelMessage } from '@erc7824/nitrolite';

// Funding channel from Unified Balance (e.g. after faucet)
const resizeMessage = await createResizeChannelMessage(sessionSigner, {
    channel_id: channelId as `0x${string}`,
    allocate_amount: 20n, // from Unified Balance into channel
    funds_destination: account.address,
});

ws.send(resizeMessage);

// On response.res[1] === 'resize_channel': submit to chain
const { channel_id, state, server_signature } = response.res[2];
const resizeState = { /* map state + channel_id + server_signature */ };
const proofStates = []; // optionally from client.getChannelData(channelId).lastValidState
const { txHash } = await nitroliteClient.resizeChannel({ resizeState, proofStates });
```

### Get ledger balances

```typescript
import { createGetLedgerBalancesMessage } from '@erc7824/nitrolite';

const ledgerMsg = await createGetLedgerBalancesMessage(sessionSigner, account.address, Date.now());
ws.send(ledgerMsg);
```

### Close channel and withdraw

Request close with `createCloseChannelMessage(sessionSigner, channelId, account.address)`. On `close_channel` response, call `nitroliteClient.closeChannel({ finalState, stateData })`, then withdraw from custody: `nitroliteClient.withdrawal(tokenAddress, withdrawableBalance)`.

### Transfer (Unified Ledger)

```typescript
import { createTransferMessage } from '@erc7824/nitrolite';

const transferMessage = await createTransferMessage(sessionSigner, {
    destination: recipientAddress as `0x${string}`,
    allocations: [{ asset: 'usdc', amount: '0.01' }],
}, Date.now());

ws.send(transferMessage);
```

### Multi-Party App Sessions

Use **NitroRPC/0.4** for new apps (intent system). App session = programmable escrow with quorum/weights. Allocations represent **final state**, not delta; version must increment by **exactly 1** per update.

**Intents (NitroRPC/0.4):** `OPERATE` (redistribute, sum unchanged), `DEPOSIT` (add from unified balance), `WITHDRAW` (remove to unified balance).

```typescript
import { 
    createAppSessionMessage,
    createSubmitAppStateMessage,
    createCloseAppSessionMessage,
    RPCAppDefinition,
    RPCProtocolVersion
} from '@erc7824/nitrolite';

const appDefinition: RPCAppDefinition = {
    protocol: RPCProtocolVersion.NitroRPC_0_4,
    participants: [address1, address2],
    weights: [50, 50],
    quorum: 100,
    challenge: 3600,
    nonce: Date.now(),
    application: 'Your App Name',
};

const allocations = [
    { participant: address1, asset: 'usdc', amount: '0.01' },
    { participant: address2, asset: 'usdc', amount: '0.00' },
];

const sessionMessage = await createAppSessionMessage(sessionSigner1, { definition: appDefinition, allocations });
ws.send(sessionMessage);

// submit_app_state: collect quorum of signatures; version = previous + 1
const updateMessage = await createSubmitAppStateMessage(sessionSigner1, { app_session_id: sessionId, allocations: newAllocations });
// Add other participants' signatures to meet quorum, then ws.send(...)

// close_app_session: quorum required; final allocations distributed to unified balances
const closeMessage = await createCloseAppSessionMessage(sessionSigner1, { app_session_id: sessionId, allocations: finalAllocations });
```

## Message Handling Pattern

Wire protocol uses **snake_case** method names in `response.res[1]`. Handle in `ws.on('message')`:

```typescript
const method = response.res?.[1];
const params = response.res?.[2];

switch (method) {
    case 'auth_challenge': /* sign challenge, send auth_verify */ break;
    case 'auth_verify': /* session ready */ break;
    case 'create_channel': /* submit to chain */ break;
    case 'resize_channel': /* submit resize */ break;
    case 'close_channel': /* submit close, then withdrawal */ break;
    case 'channels': /* list of channels */ break;
    case 'get_config': /* chains, contracts, brokerAddress */ break;
    case 'get_ledger_balances': /* balances */ break;
    case 'transfer': /* transfer result */ break;
    case 'error': /* params.error */ break;
}
```

## Contract Addresses

Prefer **get_config** for current addresses. Reference values:

**Base (mainnet):**
- custody: `0x490fb189DdE3a01B00be9BA5F41e3447FbC838b6`
- adjudicator: `0x7de4A0736Cf5740fD3Ca2F2e9cc85c9AC223eF0C`
- USDC: `0x833589fcd6edb6e08f4c7c32d4f71b54bda02913`

**Sepolia (Sandbox):**
- custody: `0x019B65A265EB3363822f2752141b3dF16131b262`
- adjudicator: `0x7c7ccbc98469190849BCC6c926307794fDfB11F2`
- ytest.usd token: `0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238`

## Best Practices

1. **Session key management**
   - One session key per wallet+application; new registration invalidates the previous one.
   - Set allowances and short expiration (e.g. 1–24h). Empty allowances = no spending.
   - Special application `"clearnode"` bypasses allowances (deprecated); expiration still applies.
   - Use `get_session_keys` to monitor usage; clear keys on logout.

2. **Error handling**
   - Handle `error` method in responses; errors are descriptive messages, not numeric codes.
   - Session expiry returns `"session expired, please re-authenticate"`; re-run auth flow.
   - Implement WebSocket reconnection and validate message shape before processing.

3. **State management**
   - Track channel states; use latest state from node for resize/close.
   - Store previous states for `resizeChannel` proof states when needed.
   - App sessions: version must increment by exactly 1 per update.

4. **Multi-party**
   - Collect quorum of signatures before sending; verify weights sum ≥ quorum.
   - Plan for disputes: set challenge period (e.g. 1–24h); allocations verifiable by third parties.

5. **Security**
   - Never commit `.env` or private keys; use env vars only.
   - Session key private key stays on device; never transmit.
   - auth_verify must be signed by **main wallet** (EIP-712), not session key.

## Common Patterns

### Utility Functions

```typescript
// lib/utils.ts
import { generatePrivateKey, privateKeyToAccount } from 'viem/accounts';
import type { Address } from 'viem';

export interface SessionKey {
    privateKey: `0x${string}`;
    address: Address;
}

export const generateSessionKey = (): SessionKey => {
    const privateKey = generatePrivateKey();
    const account = privateKeyToAccount(privateKey);
    return { privateKey, address: account.address };
};
```

### Authentication Helper

```typescript
// lib/auth.ts — wrap WebSocket + auth flow
export async function authenticateWithChallenge(ws: WebSocket, walletClient: WalletClient, authParams: AuthParams): Promise<SessionKey> {
    const sessionKey = generateSessionKey();
    const authMessage = await createAuthRequestMessage({
        address: walletClient.account?.address as `0x${string}`,
        session_key: sessionKey.address,
        application: authParams.application,
        allowances: authParams.allowances,
        expires_at: authParams.expires_at,
        scope: authParams.scope,
    });

    return new Promise((resolve, reject) => {
        const handler = (data: Buffer | string) => {
            const res = JSON.parse(data.toString());
            const method = res.res?.[1];
            const params = res.res?.[2];
            if (method === 'auth_challenge') {
                const signer = createEIP712AuthMessageSigner(walletClient, { ...authParams, session_key: sessionKey.address }, { name: authParams.application });
                createAuthVerifyMessageFromChallenge(signer, params.challenge_message).then((msg) => ws.send(msg));
            } else if (method === 'auth_verify') {
                ws.off('message', handler);
                params?.success ? resolve(sessionKey) : reject(new Error('Authentication failed'));
            } else if (method === 'error') {
                reject(new Error(params?.error ?? 'Auth error'));
            }
        };
        ws.on('message', handler);
        ws.send(authMessage);
    });
}
```

## Troubleshooting

### Authentication
- **Invalid signature**: EIP-712 must be signed by **main wallet** in auth_verify.
- **Challenge expired**: Challenge valid ~5 minutes; restart from auth_request.
- **Session key already registered**: Generate a new session keypair.
- **Session expired**: Re-authenticate (auth_request → auth_challenge → auth_verify or send existing jwt in auth_verify).
- **operation denied: insufficient session key allowance**: Session key spending cap reached; create new session with higher allowance or use main wallet.

### Channel / Resize
- **InsufficientBalance**: Using `resize_amount` without L1 custody deposit. Use `allocate_amount` to fund from Unified Balance (e.g. Sandbox faucet).
- **DepositAlreadyFulfilled**: Double-submit or channel already open/funded; check state before sending.
- **InvalidState**: Resizing closed channel or wrong version; use latest channel state from node.
- **operation denied: non-zero allocation**: Too many stale open channels; close them (e.g. run a close_all script using `client.getOpenChannels()` and `createCloseChannelMessage` + `closeChannel`).
- **Timeout waiting for User to fund Custody**: Multiple runs can accumulate required balance; close existing channels first (`npx tsx close_all.ts` or equivalent).

### Cleanup
- Use `nitroliteClient.getOpenChannels()` to list open channels; for each, send `createCloseChannelMessage` then on `close_channel` response call `client.closeChannel(...)` to settle on-chain.

## Common Issues (Environment)

- **"Module not found"**: Add `"type": "module"` in package.json and use ESM imports.
- **"Cannot find module 'viem'"**: Run `pnpm install` (or npm/yarn).
- **RPC rate limiting**: Use a dedicated RPC provider (e.g. Alchemy, Infura), not public endpoints.
- **TypeScript errors with viem**: Set `"moduleResolution": "bundler"` or `"node16"` in tsconfig.json.

## Test tokens (Sandbox)

Request test tokens (ytest.usd) to **Unified Balance** (no on-chain deposit needed):

```bash
curl -X POST https://clearnet-sandbox.yellow.com/faucet/requestTokens \
  -H "Content-Type: application/json" \
  -d '{"userAddress": "<your_wallet_address>"}'
```

## Dependencies

```json
{
  "dependencies": {
    "@erc7824/nitrolite": "^0.5.0",
    "viem": "^2.x",
    "dotenv": "^17.x"
  },
  "devDependencies": {
    "typescript": "^5.x",
    "@types/node": "^20.x",
    "tsx": "^4.x"
  }
}
```

Optional: `yellow-ts` for a higher-level WebSocket client wrapper; official quickstart uses raw WebSocket + `@erc7824/nitrolite`.

## Additional resources

- Endpoints, env vars, wire methods, resize vs allocate, troubleshooting, faucet: [reference.md](reference.md)

## Resources

- [Yellow Docs](https://docs.yellow.org/) — Learn, Build, Run a Clearnode
- [What is Yellow SDK](https://docs.yellow.org/docs/learn/introduction/what-is-yellow-sdk) — Nitrolite RPC vs NitroliteClient
- [Prerequisites & Environment](https://docs.yellow.org/docs/learn/getting-started/prerequisites) — Setup, .env, faucet, get_config
- [Quickstart](https://docs.yellow.org/docs/learn/getting-started/quickstart) — Full channel lifecycle, Sepolia, close_all
- [Key Terms](https://docs.yellow.org/docs/learn/getting-started/key-terms) — State, channel, allocation, Clearnode, unified balance
- [State Channels vs L1/L2](https://docs.yellow.org/docs/learn/core-concepts/state-channels-vs-l1-l2)
- [App Sessions](https://docs.yellow.org/docs/learn/core-concepts/app-sessions) — Quorum, intents, NitroRPC/0.4
- [Session Keys](https://docs.yellow.org/docs/learn/core-concepts/session-keys) — Allowances, expiration, v0.5.0 wallet-as-participant
- [Authentication](https://docs.yellow.org/docs/protocol/off-chain/authentication) — auth_request (public), auth_challenge, auth_verify (EIP-712)
- [Glossary](https://docs.yellow.org/docs/protocol/glossary)
- [Nitrolite / ERC-7824](https://erc7824.org/) — Protocol spec
- Endpoints: Production `wss://clearnet.yellow.com/ws`, Sandbox `wss://clearnet-sandbox.yellow.com/ws`

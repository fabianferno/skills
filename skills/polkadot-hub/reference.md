# Polkadot Hub — reference

Cheat sheet for **smart contracts, dual VM, fees, and links**. The full narrative and migration detail live in [SKILL.md](SKILL.md). **Source of truth**: [docs.polkadot.com](https://docs.polkadot.com) — verify live pages before locking production behavior.

---

## When to use which VM

| Situation | VM |
|-----------|-----|
| Drop-in Solidity, existing audits, exact EVM semantics | **REVM** |
| Heavy compute, lower fees, new Rust/Solidity→PVM (`resolc`) work | **PVM (PolkaVM)** |
| Cross-VM | Supported — EVM ↔ PVM calls |

**Compilers**: EVM bytecode → `solc` on REVM. PVM → **`resolc`** ([Revive](https://github.com/paritytech/revive)), YUL IR pipeline.

---

## Resource metering (not just “gas”)

Externally: Ethereum-style **single gas** on RPC. Internally:

| Dimension | Meaning |
|-----------|---------|
| **`ref_time`** | Compute (closest to classic gas) |
| **`proof_size`** | State proof size for validators |
| **`storage_deposit`** | Balance locked for new storage (refunded when freed) |

**Tips**: Add **10–20%** buffer on limits/prices — fee multiplier moves with block fullness. `eth_estimateGas` dry-runs deposit + execution + buffer.

---

## PVM limits (high-signal)

| Limit | Typical cap |
|-------|-------------|
| Call stack depth | **5** |
| Memory per contract | **64 KB** hard |
| Event topics | **4** |
| Event payload (incl. topics) | **416 bytes** |
| Storage value size | **416 bytes** |
| Contract code blob | **~100 KB** |

Limits are not reduced over time (compat); may increase.

---

## Critical PVM differences (migration)

1. **Reentrancy**: The **2300 gas stipend** on `send`/`transfer` does **not** protect on PVM — gas limits on cross-calls are not enforced the same way. Use **explicit reentrancy guards**.
2. **Factories**: **Upload code first**, then instantiate by **hash**. Raw runtime codegen / dynamic bytecode patterns fail without pre-upload (`CodeNotFound`).
3. **Unsupported / errors**: `selfdestruct`, `extcodecopy`, `pc`, blob opcodes — compile-time errors on PVM where listed in main skill.
4. **Randomness**: `prevrandao` / `difficulty` — **not** trustworthy randomness on PVM; use documented Polkadot approaches.
5. **Cross-contract gas**: Supplied gas often **ignored**; remaining resources **forwarded** — do not rely on callee gas caps for safety.

---

## Existential deposit (ED)

Accounts below ED are **reaped**. RPC balances are adjusted so Ethereum-facing tools keep working, but **be aware** for economics and new accounts (first transfer pays ED).

---

## Precompiles (categories)

- Ethereum-native (ecRecover, SHA256, etc.)
- **ERC-20 precompile** — native assets as ERC-20 from Solidity
- System, storage
- **XCM precompile** — cross-chain messaging from contracts

Details: [Precompiles](https://docs.polkadot.com/smart-contracts/precompiles/).

---

## Tooling quick map

| Goal | Entry point |
|------|----------------|
| Connect wallet / network | [Connect](https://docs.polkadot.com/smart-contracts/connect/) |
| Test tokens | [Faucet](https://docs.polkadot.com/smart-contracts/faucet/) |
| Local node | [Local dev node](https://docs.polkadot.com/smart-contracts/dev-environments/local-dev-node/) |
| Hardhat / Foundry / Remix | [Dev environments](https://docs.polkadot.com/smart-contracts/dev-environments/hardhat/) (siblings in nav) |
| Ethers / Viem / Wagmi | [Libraries](https://docs.polkadot.com/smart-contracts/libraries/ethers-js/) |
| EVM vs PVM deep dive | [EVM vs PVM](https://docs.polkadot.com/smart-contracts/for-eth-devs/evm-vs-pvm/) |
| Gas model | [Gas model](https://docs.polkadot.com/smart-contracts/for-eth-devs/gas-model/) |
| Parachains / SDK | [Install Polkadot SDK](https://docs.polkadot.com/parachains/install-polkadot-sdk/) |
| OpenZeppelin wizard | [wizard.openzeppelin.com/polkadot](https://wizard.openzeppelin.com/polkadot) |

---

## Related repos

- [revive (resolc)](https://github.com/paritytech/revive)
- [polkavm](https://github.com/paritytech/polkavm)
- [pallet_revive docs](https://paritytech.github.io/polkadot-sdk/master/pallet_revive/index.html)

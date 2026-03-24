---
name: polkadot-hub
description: >
  Comprehensive guide for building on Polkadot Hub — smart contracts (EVM via REVM + RISC-V via PVM),
  parachains, cross-chain interactions (XCM), and node infrastructure. Use this skill whenever the user
  mentions Polkadot, Polkadot Hub, PVM, PolkaVM, REVM on Polkadot, pallet_revive, Revive compiler,
  resolc, Polkadot smart contracts, Polkadot SDK, parachains, XCM, coretime, FRAME pallets,
  Substrate development, or any RISC-V blockchain VM topic. Also trigger for EVM-to-Polkadot migration,
  cross-chain messaging, Polkadot precompiles, or questions about deploying Solidity contracts on
  Polkadot. This skill covers everything from deploying a basic ERC-20 on Polkadot Hub to writing
  optimized PVM contracts in Rust and building full parachain runtimes.
---

# Polkadot Hub Development Skill

> **Source of truth**: https://docs.polkadot.com (last verified: March 2026)
> This skill is synthesized from a thorough crawl of the official Polkadot developer documentation.
> When in doubt, ALWAYS verify against the live docs — things move fast.

---

## Table of Contents

1. [Platform Overview](#1-platform-overview)
2. [Dual VM Architecture](#2-dual-vm-architecture)
3. [REVM Backend (EVM Compatibility)](#3-revm-backend)
4. [PVM Backend (RISC-V / PolkaVM)](#4-pvm-backend)
5. [EVM vs PVM — Critical Differences](#5-evm-vs-pvm-critical-differences)
6. [Gas Model & Resource Metering](#6-gas-model--resource-metering)
7. [Contract Deployment](#7-contract-deployment)
8. [Precompiles](#8-precompiles)
9. [Development Environments & Tooling](#9-development-environments--tooling)
10. [Libraries & SDKs](#10-libraries--sdks)
11. [Cookbook — Common Recipes](#11-cookbook--common-recipes)
12. [Parachains & Polkadot SDK](#12-parachains--polkadot-sdk)
13. [Chain Interactions & XCM](#13-chain-interactions--xcm)
14. [Node Infrastructure](#14-node-infrastructure)
15. [Key Reference Links](#15-key-reference-links)
16. [Common Gotchas & Migration Pitfalls](#16-common-gotchas--migration-pitfalls)

---

## 1. Platform Overview

Polkadot Hub is a production-ready smart contract platform that combines Ethereum compatibility with Polkadot's performance and cross-chain capabilities. It runs on the Polkadot relay chain as a system parachain.

**Two paths for developers:**
- **Smart Contracts** — Deploy directly on Polkadot Hub using Solidity or Rust. No parachain needed.
- **Parachains** — Build a custom blockchain (parachain) with the Polkadot SDK for sovereign control.

**Key selling points:**
- Full Ethereum JSON-RPC compatibility (MetaMask, Hardhat, Remix, Foundry all work)
- Dual VM: REVM for unmodified EVM contracts, PVM for optimized RISC-V execution
- Cross-VM interop: EVM contracts can call PVM contracts and vice versa
- Cross-chain composability via XCM (no bridges needed within Polkadot ecosystem)
- Native token: DOT

---

## 2. Dual VM Architecture

Polkadot Hub supports two execution backends within a single smart contract platform. Both share the same RPC interface, tooling, and precompiles.

### Architecture Flow

```
User/dApp → Ethereum JSON-RPC Proxy → Blockchain Node → pallet_revive
                                                              ↓
                                                    ┌─────────┴─────────┐
                                                    │                   │
                                                  REVM              PolkaVM
                                              (EVM bytecode)    (RISC-V bytecode)
                                                    │                   │
                                                    └─────────┬─────────┘
                                                         Execution
```

**`pallet_revive`** is the runtime module (pallet) that processes Ethereum-style transactions. The Ethereum JSON-RPC proxy repackages Ethereum transactions into Polkadot-compatible extrinsics, preserving the original payload for tool compatibility.

### When to Use Which

| Use Case | Recommended VM |
|---|---|
| Migrating existing Solidity contracts unchanged | **REVM** |
| Using audit tools that inspect EVM bytecode | **REVM** |
| Need exact EVM behavior guarantees | **REVM** |
| Computationally intensive workloads | **PVM** |
| Want lower fees via optimized execution | **PVM** |
| Building new contracts and want max performance | **PVM** |
| Writing contracts in Rust (not Solidity) | **PVM** |

---

## 3. REVM Backend

REVM is a complete Rust implementation of the Ethereum Virtual Machine (from the `revm` crate by bluealloy). It runs unmodified Ethereum contracts on Polkadot Hub.

**Key facts:**
- Zero modifications required to deploy existing Ethereum contracts
- Full EVM opcode compatibility
- Standard Ethereum tooling works: Hardhat, Foundry, Remix, MetaMask
- Libraries: Ethers.js, Web3.js, Viem, Wagmi, Web3.py
- OpenZeppelin Contracts Wizard available for Polkadot: https://wizard.openzeppelin.com/polkadot
- REVM is live on Polkadot Hub TestNet

---

## 4. PVM Backend

PVM (Polkadot Virtual Machine / PolkaVM) uses a **RISC-V instruction set** instead of the EVM stack-based architecture. Contracts compile to RISC-V bytecode for enhanced performance and lower fees.

**Compilation paths to PVM:**
- **Solidity → PVM**: Use the `resolc` compiler (Revive compiler). Solidity code, including inline assembly, compiles to RISC-V via the YUL IR pipeline.
- **Rust → PVM**: Write contracts directly in Rust. Tooling is maturing; LLMs and coding agents can assist development.

**Key facts:**
- RISC-V format bytecode (not EVM bytecode)
- `resolc` compiler: https://github.com/paritytech/revive
- On-chain constructors for contract instantiation
- 64KB memory limit per contract
- Hash-based code referencing (not raw bytecode)
- Cross-VM interop: PVM contracts can call EVM contracts and vice versa

---

## 5. EVM vs PVM — Critical Differences

### Architecture

| Feature | EVM (REVM) | PVM (PolkaVM) |
|---|---|---|
| Instruction Set | Stack-based | RISC-V |
| Bytecode Format | EVM bytecode | RISC-V format |
| Compiler | Standard `solc` | `resolc` (Revive) |
| Contract Size Limit | 24KB code size | ~100KB code blob |
| Memory Model | Indirect via gas costs | Hard 64KB limit per contract |
| Code Introspection | `EXTCODECOPY` supported | Limited; use on-chain constructors |
| Resource Metering | Single gas metric | Multi-dimensional (ref_time, proof_size, storage_deposit) |
| Call Stack Depth | Limited by gas | Max 5 |
| `selfdestruct` | Deprecated but exists | Not supported (compile error) |

### Memory Limits (PVM-specific)

| Limit | Maximum |
|---|---|
| Call stack depth | 5 |
| Event topics | 4 |
| Event data payload (incl. topics) | 416 bytes |
| Storage value size | 416 bytes |
| Transient storage variables | 128 uint values |
| Immutable variables | 16 uint values |
| Contract code blob size | ~100 kilobytes |
| Memory per contract | 64KB |

> Limits will never be decreased (backward compatibility guarantee) but may be increased.

### Reentrancy — CRITICAL WARNING

**The 2300 gas stipend that Solidity provides for `address.payable.{send,transfer}` offers NO reentrancy protection in PVM.** Gas limits are ignored in cross-contract calls because PVM forwards all remaining resources. You MUST implement explicit reentrancy guards (mutex locks) when calling untrusted contracts. The compiler tries to detect and mitigate `send`/`transfer` patterns, but do not rely on it.

### Unsupported EVM Opcodes in PVM

These produce **compile-time errors** in PVM:
- `pc` (program counter)
- `extcodecopy`
- `blobhash`, `blobbasefee` (EIP-4844 blob ops)
- `selfdestruct`

### Changed Opcode Behavior in PVM

- `prevrandao` / `difficulty` → Returns constant `2500000000000000` (not real randomness)
- `gas` / `gaslimit` → Returns only `ref_time` component
- `call` / `delegatecall` / `staticcall` → Ignores supplied gas limits, forwards all resources
- `create` / `create2` → Expects code hash + constructor args (not raw bytecode)
- `dataoffset` → Returns contract hash (not code offset)
- `datasize` → Returns 32 bytes (hash size, not code size)
- `mload` / `mstore` / `msize` / `mcopy` → 64KB buffer limit; `OutOfBound` error if exceeded
- `codesize` (in constructor) → Returns call data size, not code blob size
- `calldataload` / `calldatacopy` (in constructor) → Offset ignored, returns 0
- `codecopy` → Only in constructor code
- `invalid` → Traps but does NOT consume remaining gas
- `address.creationCode` → Returns keccak256 hash, not actual creation code

### YUL IR Pipeline

PVM processes YUL IR exclusively. All contracts exhibit `via-ir` compilation behavior. 32-bit pointer size — values above 2^32-1 will trap the contract.

---

## 6. Gas Model & Resource Metering

### Three Resource Dimensions

Unlike Ethereum's single gas metric, Polkadot Hub tracks three separate resources:

1. **`ref_time`** — Computational time (closest to traditional gas / CPU cycles)
2. **`proof_size`** — Size of state proofs validators need to verify
3. **`storage_deposit`** — Native balance locked when creating new storage entries (returned when freed)

### Gas ↔ Weight Translation

- Externally (wallets, RPC): Single gas value (Ethereum-compatible)
- Internally (runtime): `weight` = two-dimensional tuple `(ref_time, proof_size)`
- The RPC proxy maps all three dimensions into one gas value automatically
- `WeightToFee` conversion takes the maximum of the two dimensions (after coefficients)

### Gas Estimation (`eth_estimateGas`)

Performs a dry-run covering: base overhead + transaction length fees + contract execution + storage deposits + safety buffer.

### Dynamic Gas Pricing

- Fee multiplier adjusts per-block based on network congestion
- Full blocks → multiplier increases (more expensive)
- Empty blocks → multiplier decreases (cheaper)
- Similar to Ethereum's base fee mechanism
- **Tip**: Add 10-20% buffer to gas limit and gas price to handle multiplier changes

### Transaction Flow

1. Transaction enters pool
2. Gas mapped to weight; temporary hold created for max fee
3. Funds check (rejected if insufficient)
4. Contract executes within weight limits
5. Settlement: fees charged from actual weight + length fee; unused hold refunded
6. Included in block

---

## 7. Contract Deployment

### Standard Deployment (REVM)

Works exactly like Ethereum. Deploy with Hardhat, Remix, Foundry, or any standard tool.

### PVM Deployment — Two-Step Process

PVM fundamentally differs from EVM deployment:

1. **Upload code first**: All contract bytecode must be uploaded to the chain before instantiation
2. **Instantiate**: Reference the uploaded code hash to create contract instances

**Factory contracts**: The common EVM pattern where contracts dynamically create other contracts will fail with `CodeNotFound` error unless dependent code was pre-uploaded. Factory contracts must be modified to work with pre-uploaded code hashes.

**Runtime code generation**: Not supported in PVM due to RISC-V bytecode format.

### Existential Deposit (ED)

Polkadot requires an existential deposit to maintain accounts (unlike Ethereum where accounts persist indefinitely with zero balance):

- Accounts below ED are automatically deleted
- Balance queries via Ethereum RPC auto-deduct ED (reported = spendable)
- Transfers to new accounts automatically include ED (extra cost in fees)
- Contract-to-contract transfers: ED drawn from transaction signer, not sending contract
- Ethereum contracts work without modification due to these transparent adjustments

---

## 8. Precompiles

Precompiles extend smart contract functionality by exposing Polkadot-native features to Solidity contracts. Categories:

### Ethereum Native Precompiles
Standard Ethereum precompiles (ecRecover, SHA256, RIPEMD160, identity, modexp, ecAdd, ecMul, ecPairing, blake2f).

### ERC-20 Precompile
Access Polkadot native assets as ERC-20 tokens from Solidity.

### System Precompile
Access system-level information and functionality.

### Storage Precompile
Direct storage access operations.

### XCM Precompile
**Cross-chain messaging from smart contracts.** Enables:
- Token transfers to other parachains
- Remote execution on other chains
- Cross-chain composability without bridges

---

## 9. Development Environments & Tooling

### Local Development Node
Run a local Polkadot Hub node for testing. Docs: https://docs.polkadot.com/smart-contracts/dev-environments/local-dev-node/

### Supported IDEs / Frameworks

| Tool | Use Case | Docs Link |
|---|---|---|
| **Remix IDE** | Browser-based, quick prototyping | https://docs.polkadot.com/smart-contracts/dev-environments/remix/ |
| **Hardhat** | Full project framework, testing, deployment | https://docs.polkadot.com/smart-contracts/dev-environments/hardhat/ |
| **Foundry** | Fast Solidity testing & deployment | https://docs.polkadot.com/smart-contracts/dev-environments/foundry/ |

### Network Connection

Connect MetaMask or any Ethereum wallet to Polkadot Hub TestNet:
- Details at: https://docs.polkadot.com/smart-contracts/connect/
- Faucet for test tokens: https://docs.polkadot.com/smart-contracts/faucet/
- Block explorers: https://docs.polkadot.com/smart-contracts/explorers/

### Key Tools (Polkadot Ecosystem)

| Tool | Purpose |
|---|---|
| **Pop CLI** | Scaffold and manage parachain projects |
| **Zombienet** | Spin up multi-node test networks |
| **Chopsticks** | Fork and test against live chain state |
| **Moonwall** | Testing framework for Polkadot environments |
| **Omninode** | Generic parachain node binary |
| **ParaSpell** | XCM tooling for cross-chain operations |
| **Subxt** | Rust client for Substrate-based chains |
| **Polkadot-API (PAPI)** | TypeScript client for Polkadot |
| **Polkadot.js API** | JavaScript API for Substrate chains |
| **Dedot** | Lightweight TypeScript Substrate client |
| **Polkadart** | Dart SDK for Substrate |
| **Python Substrate Interface** | Python client for Substrate |
| **Sidecar REST API** | REST API for Substrate node data |
| **Light Clients** | Browser-embedded chain access (Smoldot) |

---

## 10. Libraries & SDKs

### Ethereum-Compatible Libraries (for Smart Contracts)

All work with Polkadot Hub's Ethereum RPC:

| Library | Language | Docs |
|---|---|---|
| **Ethers.js** | JavaScript/TypeScript | https://docs.polkadot.com/smart-contracts/libraries/ethers-js/ |
| **Web3.js** | JavaScript/TypeScript | https://docs.polkadot.com/smart-contracts/libraries/web3-js/ |
| **Viem** | TypeScript | https://docs.polkadot.com/smart-contracts/libraries/viem/ |
| **Wagmi** | React hooks | https://docs.polkadot.com/smart-contracts/libraries/wagmi/ |
| **Web3.py** | Python | https://docs.polkadot.com/smart-contracts/libraries/web3-py/ |

### Polkadot-Native SDKs (for Chain Interactions)

| SDK | Language | Docs |
|---|---|---|
| **Polkadot-API (PAPI)** | TypeScript | https://docs.polkadot.com/reference/tools/papi/ |
| **Subxt** | Rust | https://docs.polkadot.com/reference/tools/subxt/ |
| **Polkadot.js API** | JavaScript | https://docs.polkadot.com/reference/tools/polkadot-js-api/ |

---

## 11. Cookbook — Common Recipes

### Deploy a Basic Contract
- Via Remix: https://docs.polkadot.com/smart-contracts/cookbook/smart-contracts/deploy-basic/basic-remix/
- Via Hardhat: https://docs.polkadot.com/smart-contracts/cookbook/smart-contracts/deploy-basic/basic-hardhat/

### Deploy an ERC-20 Token
- Via Remix: https://docs.polkadot.com/smart-contracts/cookbook/smart-contracts/deploy-erc20/erc20-remix/
- Via Hardhat: https://docs.polkadot.com/smart-contracts/cookbook/smart-contracts/deploy-erc20/erc20-hardhat/

### Deploy an NFT (ERC-721)
- Via Remix: https://docs.polkadot.com/smart-contracts/cookbook/smart-contracts/deploy-nft/nft-remix/
- Via Hardhat: https://docs.polkadot.com/smart-contracts/cookbook/smart-contracts/deploy-nft/nft-hardhat/

### Build a Full DApp (Zero to Hero)
- https://docs.polkadot.com/smart-contracts/cookbook/dapps/zero-to-hero/

### Port Ethereum DApps
- Uniswap V2 port: https://docs.polkadot.com/smart-contracts/cookbook/eth-dapps/uniswap-v2/

### OpenZeppelin Contracts Wizard
- https://wizard.openzeppelin.com/polkadot — Generate ERC-20, ERC-721, and other standard contracts for Polkadot Hub

---

## 12. Parachains & Polkadot SDK

For building custom blockchains (not just smart contracts).

### Getting Started
- Install Polkadot SDK: https://docs.polkadot.com/parachains/install-polkadot-sdk/
- Set up parachain template: https://docs.polkadot.com/parachains/launch-a-parachain/set-up-the-parachain-template/
- Deploy to Polkadot: https://docs.polkadot.com/parachains/launch-a-parachain/deploy-to-polkadot/
- Obtain Coretime: https://docs.polkadot.com/parachains/launch-a-parachain/obtain-coretime/

### Customize Runtime
- Add existing pallets: https://docs.polkadot.com/parachains/customize-runtime/add-existing-pallets/
- Add smart contract support: https://docs.polkadot.com/parachains/customize-runtime/add-smart-contract-functionality/
- Create custom pallets: https://docs.polkadot.com/parachains/customize-runtime/pallet-development/create-a-pallet/
- Mock runtime for testing: https://docs.polkadot.com/parachains/customize-runtime/pallet-development/mock-runtime/
- Unit test pallets: https://docs.polkadot.com/parachains/customize-runtime/pallet-development/pallet-testing/
- Benchmark pallets: https://docs.polkadot.com/parachains/customize-runtime/pallet-development/benchmark-pallet/

### Testing
- Fork a parachain: https://docs.polkadot.com/parachains/testing/fork-a-parachain/
- Run parachain network: https://docs.polkadot.com/parachains/testing/run-a-parachain-network/

### Maintenance
- Runtime upgrades (forkless): https://docs.polkadot.com/parachains/runtime-maintenance/runtime-upgrades/
- Storage migrations: https://docs.polkadot.com/parachains/runtime-maintenance/storage-migrations/
- Coretime renewal: https://docs.polkadot.com/parachains/runtime-maintenance/coretime-renewal/

### Interoperability
- HRMP channels between parachains: https://docs.polkadot.com/parachains/interoperability/channels-between-parachains/
- HRMP with system parachains: https://docs.polkadot.com/parachains/interoperability/channels-with-system-parachains/

### EVM-Compatible Parachains (Alternatives)
- **Moonbeam**: Full Ethereum-compatible parachain, interoperability hub
- **Astar**: Dual VM (EVM + Wasm)
- **Acala**: DeFi-focused with enhanced EVM+

---

## 13. Chain Interactions & XCM

### Query On-Chain Data
- Read state with SDKs: https://docs.polkadot.com/chain-interactions/query-data/query-sdks/
- REST API: https://docs.polkadot.com/chain-interactions/query-data/query-rest/
- Runtime API calls: https://docs.polkadot.com/chain-interactions/query-data/runtime-api-calls/

### Send Transactions
- With SDKs: https://docs.polkadot.com/chain-interactions/send-transactions/with-sdks/
- Calculate fees: https://docs.polkadot.com/chain-interactions/send-transactions/calculate-transaction-fees/
- Pay fees with different tokens: https://docs.polkadot.com/chain-interactions/send-transactions/pay-fees-with-different-tokens/

### Cross-Chain (XCM)
- Transfer assets into Polkadot (from external chains): https://docs.polkadot.com/chain-interactions/send-transactions/interoperability/transfer-assets-into-polkadot/
- Transfer between parachains: https://docs.polkadot.com/chain-interactions/send-transactions/interoperability/transfer-assets-parachains/
- Estimate XCM fees: https://docs.polkadot.com/chain-interactions/send-transactions/interoperability/estimate-xcm-fees/
- Debug XCM messages: https://docs.polkadot.com/chain-interactions/send-transactions/interoperability/debug-and-preview-xcms/

### Token Operations
- Register local asset: https://docs.polkadot.com/chain-interactions/token-operations/register-local-asset/
- Register foreign asset: https://docs.polkadot.com/chain-interactions/token-operations/register-foreign-asset/
- Convert assets: https://docs.polkadot.com/chain-interactions/token-operations/convert-assets/

---

## 14. Node Infrastructure

### Run Nodes
- Polkadot Hub RPC node: https://docs.polkadot.com/node-infrastructure/run-a-node/polkadot-hub-rpc/
- Parachain RPC: https://docs.polkadot.com/node-infrastructure/run-a-node/parachain-rpc/
- Full relay chain node: https://docs.polkadot.com/node-infrastructure/run-a-node/relay-chain/full-node/

### Run a Collator
- https://docs.polkadot.com/node-infrastructure/run-a-collator/

### Run a Validator
- Requirements: https://docs.polkadot.com/node-infrastructure/run-a-validator/requirements/
- Setup: https://docs.polkadot.com/node-infrastructure/run-a-validator/onboarding-and-offboarding/set-up-validator/
- Key management: https://docs.polkadot.com/node-infrastructure/run-a-validator/onboarding-and-offboarding/key-management/
- Staking rewards: https://docs.polkadot.com/node-infrastructure/run-a-validator/staking-mechanics/rewards/
- Offenses/slashing: https://docs.polkadot.com/node-infrastructure/run-a-validator/staking-mechanics/offenses-and-slashes/

---

## 15. Key Reference Links

| Topic | URL |
|---|---|
| Full docs home | https://docs.polkadot.com |
| Smart Contracts overview | https://docs.polkadot.com/smart-contracts/overview/ |
| EVM vs PVM deep dive | https://docs.polkadot.com/smart-contracts/for-eth-devs/evm-vs-pvm/ |
| Dual VM architecture | https://docs.polkadot.com/smart-contracts/for-eth-devs/dual-vm-stack/ |
| Gas model | https://docs.polkadot.com/smart-contracts/for-eth-devs/gas-model/ |
| Contract deployment | https://docs.polkadot.com/smart-contracts/for-eth-devs/contract-deployment/ |
| Accounts (Eth → Polkadot) | https://docs.polkadot.com/smart-contracts/for-eth-devs/accounts/ |
| Blocks/Transactions/Fees | https://docs.polkadot.com/smart-contracts/for-eth-devs/blocks-transactions-fees/ |
| JSON-RPC APIs | https://docs.polkadot.com/smart-contracts/for-eth-devs/json-rpc-apis/ |
| Precompiles | https://docs.polkadot.com/smart-contracts/precompiles/ |
| XCM precompile | https://docs.polkadot.com/smart-contracts/precompiles/xcm/ |
| Connect / Network details | https://docs.polkadot.com/smart-contracts/connect/ |
| Faucet | https://docs.polkadot.com/smart-contracts/faucet/ |
| Explorers | https://docs.polkadot.com/smart-contracts/explorers/ |
| Glossary | https://docs.polkadot.com/reference/glossary/ |
| Revive compiler (resolc) | https://github.com/paritytech/revive |
| PolkaVM | https://github.com/paritytech/polkavm |
| pallet_revive API docs | https://paritytech.github.io/polkadot-sdk/master/pallet_revive/index.html |
| OpenZeppelin for Polkadot | https://wizard.openzeppelin.com/polkadot |
| Polkadot SDK | https://docs.polkadot.com/parachains/install-polkadot-sdk/ |

---

## 16. Common Gotchas & Migration Pitfalls

### For Ethereum Developers Moving to Polkadot Hub

1. **Reentrancy**: Gas stipends do NOT protect you on PVM. Always use reentrancy guards.
2. **Factory contracts**: Pre-upload all dependent contract code. `create`/`create2` expect code hashes, not raw bytecode on PVM.
3. **`selfdestruct`**: Not supported. Remove from contracts before migrating.
4. **`EXTCODECOPY`**: Not supported on PVM. Use on-chain constructors instead.
5. **Randomness**: `block.difficulty`/`prevrandao` returns a constant on PVM. Use Polkadot-native randomness or VRFs.
6. **Memory**: PVM has a hard 64KB limit per contract. No gradual gas cost increase — you just hit a wall.
7. **Existential deposit**: Accounts with zero balance get deleted. The RPC layer handles this transparently, but be aware.
8. **Gas estimation**: Add 10-20% buffer. Fee multiplier can change between estimation and execution.
9. **`send`/`transfer`**: Deprecated. Use `call` with reentrancy guards.
10. **Bytecode inspection tools**: Will fail on PVM contracts (RISC-V, not EVM bytecode).
11. **Constructor code**: In PVM, deploy code = runtime code (single blob). `codesize` in constructor returns call data size.
12. **Cross-contract gas limits**: Ignored on PVM. All remaining resources are forwarded.
13. **Storage deposits**: New storage entries require a deposit (refundable). This is separate from gas.
14. **Event limits**: Max 4 topics, max 416 bytes payload including topics.
15. **Transient storage**: Max 128 uint values on PVM.

### Quick Decision: REVM or PVM?

- **Just want it to work, no changes**: → REVM
- **Need bytecode audit tool compatibility**: → REVM
- **Want lower fees on compute-heavy operations**: → PVM
- **Building new from scratch, want max performance**: → PVM
- **Writing Rust contracts**: → PVM (only option)
- **Need exact EVM semantics guaranteed**: → REVM
- **Mixing both**: Totally fine — cross-VM calls work. Use REVM for standard stuff, PVM for heavy compute.

---

## Additional resources

- VM choice, metering, PVM limits, migration gotchas, tooling links: [reference.md](reference.md)

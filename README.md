`Gno.land` · `Smart Contracts` · `Infrastructure`

# Gno Infrastructure Stack

Composable on-chain primitives for Gno.land.

## Overview

Gno.land lacks reusable infrastructure. Every project that needs access control builds its own. Every team that splits revenue writes a custom solution. Every contract upgrade is announced ad-hoc — or not at all. The result is duplicated logic, inconsistent security, and an ecosystem where composability remains theoretical.

This repository indexes five focused realms that each solve one infrastructure problem. They work independently or compose together. Each is a standalone Gno package with no external dependencies, clean APIs, and full on-chain queryability.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Your Realm / DAO                   │
├──────────┬──────────┬──────────┬──────────┬─────────┤
│ fee_split│permission│ service  │ upgrade  │timelock │
│          │_registry │_registry │_registry │_guardian│
├──────────┼──────────┼──────────┼──────────┼─────────┤
│ Revenue  │  Access  │Discovery │ Upgrade  │Security │
│  Flow    │ Control  │          │ Tracking │         │
└──────────┴──────────┴──────────┴──────────┴─────────┘
                        Gno.land
```

```
service_registry   → discovery layer      — what contracts exist
permission_registry → access control layer — who can do what
timelock_guardian   → security layer       — enforce delays on critical ops
upgrade_registry   → upgrade tracking     — where did this contract move
fee_split          → value flow layer     — how revenue is distributed
```

The layers are independent but aware of each other. A realm can query `permission_registry` for access control, schedule upgrades through `timelock_guardian`, record the migration in `upgrade_registry`, register the new version in `service_registry`, and route protocol fees through `fee_split` — all without any of the five realms depending on each other.

## Components

### [fee_split](https://github.com/SillyZir/fee_split) — Revenue layer

Percentage-based revenue splitting with pull-based claims. Create a split with recipients and basis-point shares, deposit funds, recipients claim proportionally. Supports freezing for immutable configurations and rounding-dust protection.

### [permission_registry](https://github.com/SillyZir/permission_registry) — Access control layer

Shared permission management with named capabilities. Register a resource, define permissions like `"mint"` or `"pause"`, grant them to addresses. Any realm can call `Has()` to enforce access without coupling to internal state.

### [service_registry](https://github.com/SillyZir/service_registry) — Discovery layer

On-chain directory of contracts with type, description, and metadata. Enables programmatic service discovery — aggregators, frontends, and tooling can enumerate available contracts by type without off-chain coordination.

### [upgrade_registry](https://github.com/SillyZir/upgrade_registry) — Upgrade tracking layer

Contract migration chain tracking. When a contract is deprecated, the owner records its successor. `GetLatest()` resolves the full chain to the current version. Circular reference protection is enforced at write time.

### [timelock_guardian](https://github.com/SillyZir/timelock_guardian) — Security layer

Mandatory delay enforcement for sensitive operations. Actions are scheduled with a minimum waiting period, publicly visible during the delay, and executable by anyone after it elapses. Only the creator can cancel.

## Example Flow

A DAO deploying a token protocol uses all five:

```
1. Register the contract in service_registry
   → ecosystem tools can discover it by type

2. Define roles in permission_registry
   → grant "mint" to the minter, "pause" to the guardian, "upgrade" to the multisig

3. Schedule a parameter change via timelock_guardian
   → 48-hour delay, visible to all holders

4. After the delay, execute the change
   → record the migration in upgrade_registry so old integrations resolve to the new address

5. Protocol fees flow through fee_split
   → 60% to treasury, 25% to contributors, 15% to validators
   → each party claims independently
```

A project can use one component or all five. There are no mandatory dependencies between them.

## Why This Matters

**Standardization.** When projects share the same permission model, upgrade tracking, and revenue splitting, tooling works everywhere instead of being built per-project.

**Composability.** A frontend reads `service_registry` to populate available integrations. A monitoring service watches `timelock_guardian` for pending critical actions. An auditor traces upgrade history through `upgrade_registry`. None of this requires custom integration per project.

**Reduced duplication.** Access control logic written once, reviewed once, used everywhere. Revenue splitting that doesn't need to be re-audited for every new DAO.

**Safer contracts.** Shared infrastructure accumulates review and testing across the entire ecosystem. A bug found in `permission_registry` gets fixed for every project that uses it.

## Stack

- [Gno](https://gno.land) — Go-like smart contract language
- [Gno.land](https://gno.land) — Layer 1 blockchain
- No external dependencies beyond the Gno standard library

---

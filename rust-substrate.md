# Checklist — Rust-Based Chains (Substrate/FRAME & similar)

Generic checklist for reviewing Rust blockchain runtime code. Not tied to any specific target.

## Arithmetic & overflow
- [ ] All arithmetic on balances/supply uses `checked_*` or `saturating_*` — never raw `+`/`-`/`*`
- [ ] `overflow-checks` isn't silently disabled in release profile for financial code paths
- [ ] No dangerous truncating casts (`as u32`, `as u64`) on user-influenced values

## Panics & DoS
- [ ] No `.unwrap()`/`.expect()`/`panic!()` reachable from user-controlled input in runtime/extrinsic code
- [ ] Deserialization of untrusted input (extrinsics, RPC, peer messages) handles errors explicitly, never panics
- [ ] Size limits enforced on inputs before decoding, to prevent allocation-based DoS

## Unsafe code
- [ ] Every `unsafe` block has a `// SAFETY:` comment justifying soundness
- [ ] No raw pointer or transmute usage in cryptographic primitives without independent review

## Permissions & extrinsics (FRAME-specific, generalizes to similar frameworks)
- [ ] Every callable extrinsic validates origin correctly (`ensure_signed`, `ensure_root`, custom origin checks)
- [ ] Declared weights reflect real computational cost — underpriced extrinsics are a spam/DoS vector
- [ ] Storage growth from user input is bounded (no unbounded `StorageMap` growth)
- [ ] Logic running every block (`on_initialize`/`on_finalize`) is cheap and bounded

## Minting / supply integrity
- [ ] Single source of truth for total supply; every increment is traceable
- [ ] Reward/mint functions can't be triggered twice for the same event within or across blocks
- [ ] No race condition between extrinsics executed in the same block that affects accounting

## Cryptography & signatures
- [ ] Signature/hash comparisons are constant-time, not `==` on raw bytes
- [ ] No nonce reuse or predictable randomness in signature generation
- [ ] Signature and key lengths validated before deserialization

## Dependencies
- [ ] `cargo audit` run against `Cargo.lock` for known RustSec advisories
- [ ] No duplicate/conflicting versions of security-critical crates (`cargo tree --duplicates`)

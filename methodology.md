# Smart Contract / Protocol Audit Methodology

A general framework for approaching a security review of a smart contract system or blockchain protocol, independent of any specific target.

## 1. Scoping & context gathering

Before touching code:

- Read the docs, whitepaper, and any prior audit reports for the target. Prior audits tell you what's already been found (out of scope for rewards) and often hint at the team's blind spots.
- Understand the **economic model**: what has value, who can move it, and under what conditions. Most critical bugs live at the boundary between code logic and economic assumptions.
- Map the **trust boundaries**: which actors are trusted (admin, oracle, sequencer), which are adversarial (any external caller), and which sit in between (governance, multisigs with delay).
- Identify what's explicitly in scope vs. out of scope, and note anything flagged as a "known issue" — don't waste cycles re-reporting it.

## 2. Recon & surface mapping

- Enumerate all entry points: public/external functions, extrinsics, RPC endpoints, upgrade paths.
- Build a call graph — what can call what, and under which permission level.
- Identify all state that represents value (balances, supply counters, staking positions) and trace every path that can mutate it.
- Diff against previous audited versions if available — new code since the last audit is disproportionately where new bugs live.

## 3. Automated analysis

Run automated tools first to cover ground quickly, then use their output to guide manual review — not as a substitute for it.

- **Static analysis**: linters and pattern-matchers (Slither for Solidity, clippy + custom greps for Rust) to catch known anti-patterns fast.
- **Fuzzing**: invariant/property-based fuzzing (Medusa, Echidna, cargo-fuzz) to stress state transitions against defined invariants — "total supply never decreases outside burn", "no function should ever be callable by both role A and role B", etc.
- **Symbolic execution**: tools like Halmos to explore paths automated fuzzing might miss, especially around arithmetic bounds and access control conditions.

## 4. Manual review

This is where most critical, business-logic bugs are actually found — tools rarely catch reasoning errors.

- Re-derive each function's intended invariant from the docs, then check whether the code actually enforces it in every code path, including error/edge branches.
- Pay special attention to: unit conversions, rounding direction (who benefits from truncation?), order-of-operations in multi-step transactions, and anything involving external calls or cross-contract/cross-chain state.
- Think adversarially about sequencing: what happens if this function is called twice in the same block? In an unexpected order relative to another function? With a zero, max, or negative-equivalent value?

## 5. Proof of concept

- Every finding needs a reproducible PoC — this is now table stakes for nearly every bounty program.
- Write the PoC as a test that demonstrates concrete impact (funds moved, invariant broken), not just "the assert fails."
- Keep PoCs minimal — a smaller reproduction is both more convincing and easier for the triage team to verify quickly.

## 6. Reporting

- State impact in terms the program can act on: what's the worst realistic outcome, and what preconditions does an attacker need?
- Include a suggested fix or mitigation direction — it speeds up triage and signals depth of understanding.
- Map to a standard classification (CVSS, or the platform's own severity rubric) so the ask is easy to evaluate.

## Recurring high-value bug categories

Across protocols, a disproportionate share of critical findings cluster around:

- Minting/burning logic and any place total supply is computed
- Access control gaps on privileged functions (missing role checks, wrong origin checks)
- Arithmetic — overflow/underflow, truncation direction, unit mismatches
- Reentrancy and cross-function/cross-contract state inconsistency
- Signature/message verification (replay, malleability, missing domain separation)
- Upgrade and initialization logic (uninitialized proxies, storage collisions)

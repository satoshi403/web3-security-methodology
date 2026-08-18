# Educational Writeup — Reentrancy: The Pattern Behind One of DeFi's Most Persistent Bug Classes

> This is an educational breakdown of a well-known, publicly documented vulnerability class. It does not reference any active, private, or embargoed bug bounty target.

## The core idea

Reentrancy happens when a contract makes an external call to another contract or address *before* it finishes updating its own state. If the external call can call back into the original contract, it can do so while the original function's state changes are still incomplete — effectively catching the contract "mid-thought."

The classic shape:

```solidity
function withdraw(uint amount) external {
    require(balances[msg.sender] >= amount);
    (bool success, ) = msg.sender.call{value: amount}(""); // external call FIRST
    require(success);
    balances[msg.sender] -= amount; // state updated AFTER
}
```

If `msg.sender` is a contract with a `receive()` function that calls `withdraw` again, it can re-enter before `balances[msg.sender]` is decremented — draining far more than its actual balance.

## Why it keeps happening

- Solidity's early design made external calls easy to write and easy to place in the wrong order
- Multi-contract systems (vaults, routers, hooks) reintroduce the same pattern at a *system* level even when individual contracts are locally safe — this is often called cross-contract or cross-function reentrancy, and it's much harder to spot in review
- Newer variants — **read-only reentrancy** — don't even need to steal funds directly. They exploit the fact that a view function can return stale/inconsistent state mid-reentrancy, tricking a *second, otherwise-safe* protocol that trusts that view function's output

## How to actually catch it during review

1. **Trace every external call** (`.call`, `.transfer`, `.send`, any interface call to an untrusted address) and ask: what state is still unwritten at this point?
2. **Follow the money graph across contracts**, not just within one file — a protocol's own contract might be safe in isolation but expose a reentrant window to any integrator relying on its intermediate state.
3. **Fuzz for it directly**: invariant fuzzers can be given a property like "total assets in ≥ total assets out across any call sequence" and will often find reentrancy paths humans miss, especially in multi-function state machines.
4. **Don't stop at the classic pattern** — check view functions used by *other* contracts as oracles. If they can return a mid-transaction, not-yet-settled value, that's the read-only variant.

## The fix pattern (Checks-Effects-Interactions)

```solidity
function withdraw(uint amount) external {
    require(balances[msg.sender] >= amount);
    balances[msg.sender] -= amount; // Effects: state updated FIRST
    (bool success, ) = msg.sender.call{value: amount}(""); // Interactions: external call LAST
    require(success);
}
```

Combined with a `nonReentrant` modifier as defense in depth — CEI ordering should be the primary fix, the guard is a backstop, not a substitute for it.

## Takeaway

Reentrancy isn't a "Solidity 2016 problem" — it's a general lesson about ordering state changes relative to loss of control flow, and it resurfaces in new forms (read-only reentrancy, cross-contract reentrancy in DeFi composability) every time the ecosystem adds a new layer of contracts calling contracts. The pattern to look for never really changes: **where does control leave your contract, and what haven't you finished writing yet when it does.**

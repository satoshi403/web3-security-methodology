# Checklist — EVM / Solidity Smart Contracts

Generic vulnerability checklist for reviewing Solidity contracts. Not tied to any specific target.

## Access control
- [ ] Every privileged function checks `msg.sender` against the correct role/owner — no missing modifiers
- [ ] Role changes (`transferOwnership`, `grantRole`) can't be front-run or left in a half-applied state
- [ ] `tx.origin` is never used for authorization
- [ ] Initializers on upgradeable contracts can only be called once, and can't be re-triggered via delegatecall

## Arithmetic & accounting
- [ ] All arithmetic on balances/supply uses checked math (Solidity ≥0.8 default, or SafeMath on older versions)
- [ ] Rounding direction favors the protocol, not the user, in every division
- [ ] No mismatched decimals/units between tokens in the same calculation
- [ ] Total supply invariant holds across every mint/burn path — no path exists where supply changes untracked

## Reentrancy & external calls
- [ ] Checks-Effects-Interactions pattern followed: state updated before external calls
- [ ] `nonReentrant` guards on any function making external calls that could re-enter
- [ ] Cross-function reentrancy considered, not just same-function
- [ ] Read-only reentrancy: view functions relied on by other contracts return consistent state mid-call

## Token & transfer logic
- [ ] Fee-on-transfer and rebasing tokens don't break accounting assumptions (received amount ≠ sent amount)
- [ ] `transfer`/`transferFrom` return values are checked (some tokens don't revert on failure)
- [ ] No unbounded loops over user-controlled arrays that could hit gas limits (DoS)

## Signatures & verification
- [ ] Signature verification includes domain separation (EIP-712) to prevent cross-contract replay
- [ ] Nonces or other replay protection on every signed message
- [ ] No signature malleability (raw ECDSA `s` value not bounded to lower half)

## Oracle & price feeds
- [ ] No reliance on spot AMM price as an oracle without manipulation resistance (TWAP, external oracle)
- [ ] Stale price data is rejected (heartbeat/staleness checks on oracle feeds)

## Upgradeability & storage
- [ ] Storage layout compatibility checked between implementation versions
- [ ] No storage slot collisions between proxy and implementation
- [ ] Upgrade function properly access-controlled and, ideally, timelocked

## Governance
- [ ] Proposal execution can't be manipulated by flash-loaned voting power
- [ ] Timelock delays are actually enforced on every sensitive parameter change

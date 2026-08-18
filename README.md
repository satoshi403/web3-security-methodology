Web3 Security Research — Methodology

A public reference for how I approach smart contract and blockchain security research: frameworks, checklists, and educational writeups.

This repo does not contain details of active, unresolved, or under-embargo findings. Everything here is either generic methodology or based on publicly disclosed, already-patched vulnerabilities used for educational purposes.

Contents
methodology.md — general framework for approaching a smart contract / protocol audit, from recon to report
checklists/smart-contracts.md — vulnerability checklist for EVM/Solidity contracts
checklists/rust-substrate.md — vulnerability checklist for Rust-based chains (Substrate/FRAME and similar)
writeups/ — educational breakdowns of publicly known vulnerability patterns (no proprietary or embargoed content)
Philosophy

Security research in Web3 is equal parts automation (fuzzing, static analysis, invariant testing) and judgment (understanding what a protocol is actually supposed to guarantee, and where that guarantee quietly breaks). Tools find anomalies; humans find bugs.

Responsible disclosure

All research referenced here follows the disclosure policy of the relevant bug bounty platform (HackerOne, Bugcrowd, Intigriti, HackenProof, Immunefi, Code4rena). No unresolved or embargoed vulnerability details are published in this repository.

Contact

Active on HackerOne, Bugcrowd, Intigriti, and HackenProof. Open to collaboration on audits and research.

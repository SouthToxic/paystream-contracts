# Mainnet Launch Checklist

Single source of truth for the PayStream mainnet go-live. Every item must be
checked, have a named owner, and meet its completion criteria before the
launch is announced. Related: [mainnet.md](mainnet.md),
[operations-runbook.md](operations-runbook.md),
[security/admin-multisig.md](security/admin-multisig.md).

**Sign-off:** this checklist must be reviewed and approved by at least two
team members before launch (see [Approvals](#approvals)).

## 1. Security

- [ ] **Audit completion**
  - Owner: ______
  - Completion criteria: final audit report received; all Critical/High
    findings fixed and re-verified by the auditor; Medium/Low findings fixed
    or documented as accepted risk; report linked from `SECURITY.md`.
- [ ] **LOW-02 resolution (re-initialization guard)**
  - Owner: ______
  - Completion criteria: `initialize` panics with `already initialized` on a
    second call; covered by a passing test; fix included in the release WASM.
- [ ] **Multisig admin setup**
  - Owner: ______
  - Completion criteria: mainnet admin is a multisig account (2-of-3 or
    stronger) configured per [admin-multisig.md](security/admin-multisig.md);
    signer keys held by distinct people on separate devices; `admin()`
    returns the multisig address after `propose_admin` / `accept_admin`.

## 2. Build and verification

- [ ] **WASM hash verification**
  - Owner: ______
  - Completion criteria: release WASMs built reproducibly from the tagged
    commit; SHA-256 hashes independently reproduced by a second team member;
    on-chain installed WASM hash matches the published hash.

## 3. Testing

- [ ] **Testnet smoke test**
  - Owner: ______
  - Completion criteria: the exact release WASM deployed to testnet;
    initialize, create_stream, withdraw, top_up, pause/resume, cancel, and
    upgrade flows executed successfully; results recorded in the release PR.

## 4. Deployment and publication

- [ ] **Contract ID publication**
  - Owner: ______
  - Completion criteria: mainnet stream and token contract IDs published in
    `README.md`, [mainnet.md](mainnet.md), and the release notes, with
    Stellar Expert links; IDs cross-checked by a second team member.
- [ ] **DNS / contact updates**
  - Owner: ______
  - Completion criteria: website/DNS records point to mainnet frontends;
    `stellar.toml` (if used) updated; security contact in `SECURITY.md` and
    support channels verified reachable.

## Approvals

| Reviewer | Role | Date | Approved |
|---|---|---|---|
| ______ | ______ | ______ | ☐ |
| ______ | ______ | ______ | ☐ |

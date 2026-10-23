# Operations Runbook

This runbook covers routine and emergency operations for the streaming contract. All admin actions on mainnet require multi-sig authorization (see `docs/security/admin-multisig.md`).

## Admin Multi-Sig Requirement (Mainnet)

On mainnet the admin address MUST be a 2-of-3 multi-sig (Stellar native multisig or a deployed multisig contract). No single key may hold admin authority. Every admin call below must be signed by at least two of the three designated signers.

### Signing steps for every admin call

1. **Prepare** — the operator builds the admin transaction (e.g. `set_admin`, `pause`, `unpause`, `upgrade`, parameter changes) and records the XDR.
2. **Distribute** — share the unsigned XDR with the other signers out-of-band (encrypted channel).
3. **Collect signatures** — each signer independently verifies the transaction contents (contract ID, function, arguments, network passphrase) and signs. At least **2 of 3** signatures are required.
4. **Submit** — a signer submits the fully-signed transaction to the network.
5. **Verify on-chain** — confirm the transaction succeeded and the resulting state change (e.g. new admin, paused flag) via a ledger explorer or `stellar contract invoke --id <contract> -- get_admin`.
6. **Record** — log the transaction hash, signers, and timestamp in the operations log.

> A single-signer admin transaction MUST be rejected during review. If only one signature is available, do not submit.

## Routine Operations

### Pause / Unpause

- Build the `pause` / `unpause` transaction.
- Follow the multi-sig signing steps above (2-of-3).
- Verify the paused state on-chain before announcing.

### Parameter Updates

- Build the parameter-change transaction.
- Follow the multi-sig signing steps above (2-of-3).
- Verify the new parameter values on-chain.

## Emergency Operations

For emergency drains and incident response, follow `docs/security/emergency-drain.md`. Emergency admin calls are still subject to the 2-of-3 multi-sig requirement; pre-stage signer availability so emergency actions are not blocked.

## Admin Transfer

To rotate or transfer admin authority:

1. Create the new 2-of-3 multi-sig account (see onboarding checklist in `docs/security/admin-multisig.md`).
2. Build the `set_admin` transaction targeting the new multi-sig address.
3. Collect 2-of-3 signatures from the **current** admin signers and submit.
4. Verify the new admin on-chain (`get_admin` returns the new multi-sig address).
5. Confirm the old signers can no longer authorize admin calls.

## Verification Checklist

- [ ] Admin address is a 2-of-3 multi-sig (verified on-chain)
- [ ] All admin transactions carry at least 2 signatures
- [ ] Transaction hashes and signers logged
- [ ] State changes verified on-chain after each admin call
- [ ] Procedure rehearsed on testnet before mainnet deployment

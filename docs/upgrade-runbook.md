# Upgrade Runbook: Contract WASM Upgrades

Operational runbook for the time-locked `propose_upgrade` → `execute_upgrade` flow of the stream contract. For Soroban SDK version bumps, see [upgrade-guide.md](upgrade-guide.md) and [soroban-sdk-upgrade.md](soroban-sdk-upgrade.md).

**Key constants**

| Constant | Value | Ledgers (≈5 s each) |
|---|---|---|
| `TIMELOCK_DELAY` | 172 800 s (48 h) | ≈ 34 560 |

Every admin call consumes the current admin nonce. Read it with `admin_nonce` before each step. A wrong nonce fails with E009.

```bash
export NET=testnet ID=<STREAM_CONTRACT_ID> ADMIN=<admin-identity>
nonce() { stellar contract invoke --id $ID --source $ADMIN --network $NET -- admin_nonce; }
```

## Normal upgrade

| # | Step | Duration |
|---|---|---|
| 1 | Build and upload new WASM | ~5 min (1–2 ledgers to upload) |
| 2 | `propose_upgrade` | 1 ledger |
| 3 | Wait for the timelock | 48 h (≈34 560 ledgers) |
| 4 | `execute_upgrade` | 1 ledger |
| 5 | `migrate` + verify `version` | 1–2 ledgers |
| 6 | Smoke tests | ~15 min |

1. **Build and upload.** Bump `CONTRACT_VERSION` first.
   ```bash
   cargo test && stellar contract build
   HASH=$(stellar contract upload --wasm target/wasm32v1-none/release/paystream_stream.wasm \
            --source $ADMIN --network $NET)
   ```
   Record `HASH` and the git commit in the release notes.
2. **Propose.** Record the nonce you use and the `unlock_time` from the `upgrade_proposed` event.
   ```bash
   stellar contract invoke --id $ID --source $ADMIN --network $NET -- \
     propose_upgrade --new_wasm_hash $HASH --nonce $(nonce)
   ```
   Announce the upgrade, the hash, and the unlock time to integrators.
3. **Wait** until ledger timestamp ≥ `unlock_time`. Calling `execute_upgrade` earlier fails with E026.
4. **Execute.**
   ```bash
   stellar contract invoke --id $ID --source $ADMIN --network $NET -- execute_upgrade --nonce $(nonce)
   ```
   Confirm the `upgrade_executed` event carries the expected hash.
5. **Migrate and verify.**
   ```bash
   stellar contract invoke --id $ID --source $ADMIN --network $NET -- migrate --admin <ADMIN_ADDRESS>
   stellar contract invoke --id $ID --network $NET -- version   # must equal new CONTRACT_VERSION
   ```
6. **Smoke tests** (testnet first, then mainnet): `get_stream` on an existing stream, `claimable`, `stream_count`, `is_paused`. Then on testnet run a full create → withdraw → cancel cycle.

## Rollback

- **Before execution:** cancel. This clears `PendingUpgrade` and emits `upgrade_cancelled`.
  ```bash
  stellar contract invoke --id $ID --source $ADMIN --network $NET -- cancel_upgrade --nonce $(nonce)
  ```
- **After execution:** there is no instant revert. Propose the **previous WASM hash** (it is still uploaded on-chain) and follow the normal flow, including the 48 h timelock. Pause the contract while you wait if the new version is harmful. Storage is preserved across upgrades. If the new version wrote incompatible storage, the rollback WASM must be able to read it.

## Emergency upgrade

The timelock cannot be bypassed. To limit damage:

1. **Pause first:** `pause_contract --nonce $(nonce)`. This blocks state-changing entry points within 1 ledger.
2. Upload the fixed WASM and `propose_upgrade` right away.
3. Communicate the incident and the unlock time.
4. After 48 h: `execute_upgrade` → `migrate` → verify `version` → smoke tests.
5. `unpause_contract --nonce $(nonce)` once verified.

See also: [operations-runbook.md](operations-runbook.md), [security/key-rotation.md](security/key-rotation.md).

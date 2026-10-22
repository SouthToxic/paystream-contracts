# Stellar & Soroban Primer for Non-Stellar Developers

If you come from Ethereum or another EVM chain, a few Soroban concepts work differently from what you're used to. Read this before you write your first PayStream change. For PayStream-specific terms, see the [glossary](glossary.md).

## 1. Ledgers and timestamps, not blocks and `block.timestamp`

| EVM | Stellar / Soroban |
|---|---|
| Block | **Ledger**: closes roughly every 5 seconds |
| `block.number` | `env.ledger().sequence()` (`u32`) |
| `block.timestamp` | `env.ledger().timestamp()` (`u64`, Unix seconds) |

- Ledger close time is set by validator consensus, and it never goes backwards.
- PayStream accrues tokens per **second** using `env.ledger().timestamp()`. It does not count ledgers (see [ADR-002](adr/ADR-002-per-second-accrual.md)).
- Ledger counts *are* used for storage TTLs and for the per-ledger create rate limit.
- Rule of thumb: 1 day ≈ 17 280 ledgers, 1 year ≈ 6 307 200 ledgers.

📖 [Ledgers](https://developers.stellar.org/docs/learn/fundamentals/stellar-data-structures/ledgers)

## 2. Stroops and decimals

- The native asset XLM has **7 decimals**. The smallest unit is a **stroop**: 1 XLM = 10 000 000 stroops.
- Contract amounts are always integers in the token's smallest unit (`i128`, see [ADR-001](adr/ADR-001-i128-amounts.md)). Do not assume 18 decimals the way ERC-20 code often does.
- Example: `rate_per_second = 1_000` on an XLM stream pays 0.0001 XLM/s.
- Fees are also charged in stroops (base fee 100 stroops, plus Soroban resource fees).

📖 [Lumens](https://developers.stellar.org/docs/learn/fundamentals/lumens) · [Fees](https://developers.stellar.org/docs/learn/fundamentals/fees-resource-limits-metering)

## 3. Soroban storage types

Unlike EVM storage, every Soroban entry has rent (a **TTL** in ledgers). Each entry lives in one of three buckets:

| Type | Lifetime | Use in PayStream |
|---|---|---|
| **Instance** | Shares the contract instance's TTL; loaded on every call | Small global config: `Admin`, `Paused`, `AdminNonce`, `PendingUpgrade` |
| **Persistent** | Own TTL; *archived* (restorable) when it expires | Streams, indexes, receipts. Extended on access to 1–2 years |
| **Temporary** | Own TTL; **deleted permanently** on expiry | Per-ledger rate-limit counters, archived streams |

Keep instance storage small, because all of it is loaded on every invocation. Full map: [storage-layout.md](storage-layout.md).

📖 [State archival](https://developers.stellar.org/docs/learn/encyclopedia/storage/state-archival) · [Storing data](https://developers.stellar.org/docs/build/guides/storage)

## 4. SAC: Stellar Asset Contract

Classic Stellar assets (including native XLM and issued assets like USDC) don't need a custom token contract. Each one has a built-in **Stellar Asset Contract** with a deterministic contract address. PayStream streams native XLM by passing the XLM SAC address as `token`.

```bash
stellar contract id asset --asset native --network testnet
```

📖 [Stellar Asset Contract](https://developers.stellar.org/docs/tokens/stellar-asset-contract)

## 5. SEP-41 token interface

SEP-41 is Soroban's token standard, roughly the counterpart of ERC-20: `transfer`, `transfer_from`, `approve`, `allowance`, `balance`, `decimals`, `name`, `symbol`, `burn`. SACs implement it, and so does PayStream's `contracts/token`. Differences from ERC-20:

- Authorization uses `require_auth()` instead of `msg.sender`, and the signer authorizes the exact invocation tree.
- `approve` takes an `expiration_ledger`, so allowances expire.
- Amounts are `i128`.

📖 [SEP-41](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0041.md) · [Token interface](https://developers.stellar.org/docs/tokens/token-interface) · PayStream notes: [sep41-compliance.md](sep41-compliance.md)

## Common mistakes

- Using ledger sequence as a clock. Use `timestamp()` for durations.
- Assuming 18 decimals.
- Storing unbounded per-user data in instance storage.
- Forgetting to extend the TTL on persistent entries.
- Expecting `msg.sender`. Pass the `Address` explicitly and call `require_auth()`.

📖 [Soroban docs home](https://developers.stellar.org/docs/build/smart-contracts/overview)

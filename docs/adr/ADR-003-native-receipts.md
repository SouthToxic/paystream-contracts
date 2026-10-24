# ADR-003: Native receipts instead of a separate NFT contract

- **Status:** Accepted

## Context
Employees may want to transfer the right to a stream's future payments. EVM protocols usually mint an ERC-721 for this. Soroban has no canonical NFT standard.

## Decision
Receipt ownership is stored natively in the stream contract (`DataKey::ReceiptOwner(u64)`) and moved via `transfer_receipt`. When the key is absent, ownership falls back to `stream.employee`. See [nft-receipts.md](../nft-receipts.md).

## Alternatives considered
- **Separate NFT contract**: needs cross-contract calls on every withdraw, a second deployment and upgrade path, and keeping two contracts' state in sync. It would also mean picking a non-standard NFT interface.

## Consequences
- One contract, atomic ownership checks, lower fees.
- Receipts won't show up in generic NFT wallets/marketplaces. A wrapper contract could be added later.
- Legacy streams work unchanged thanks to the fallback.

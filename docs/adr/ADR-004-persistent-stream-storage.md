# ADR-004: Persistent storage for streams

- **Status:** Accepted

## Context
The number of streams is unbounded, and each must survive for months or years.

## Decision
Each stream lives in **persistent** storage under `DataKey::Stream(id)`. Its TTL is extended on every access (`TTL_THRESHOLD` ≈ 1 year → `TTL_EXTEND_TO` ≈ 2 years). Instance storage holds only small global config.

## Alternatives considered
- **Instance storage**: the whole instance is loaded and charged on *every* invocation. Storing streams there would make fees grow with the total number of streams and eventually hit entry size limits.
- **Temporary storage**: deleted on expiry, which is unacceptable for funds. It is used only for archived streams after the 30-day wait, and for rate-limit counters.

## Consequences
- Per-stream cost is constant, and entries are independently archivable/restorable.
- Every persistent accessor must extend the TTL (pattern in `storage.rs`).
- Indexers should keep their own history, because expired entries are archived (see [storage-layout.md](../storage-layout.md)).

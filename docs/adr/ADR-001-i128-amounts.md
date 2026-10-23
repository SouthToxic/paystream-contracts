# ADR-001: Use `i128` for token amounts

- **Status:** Accepted

## Context
Stream amounts (`deposit`, `withdrawn`, `rate_per_second`, `pending_accrual`) have to interoperate with token contracts.

## Decision
All token amounts are `i128`.

## Alternatives considered
- **`u128`**: cannot represent negative values, but SEP-41 and every SAC use `i128`. Each token call would need a lossy or panicking cast.
- **`u64`**: max ≈ 1.8e19. That is too small for 18-decimal tokens, and `rate × elapsed` products would overflow.

## Consequences
- Matches SEP-41 `transfer`/`balance`, so no conversions at token boundaries.
- Negative values are invalid input and must be rejected (E001/E002 validation). Arithmetic uses `checked_*` and clamps to ≥ 0 (see `claimable_amount`).

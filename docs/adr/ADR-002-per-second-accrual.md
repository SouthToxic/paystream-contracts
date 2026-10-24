# ADR-002: Per-second accrual instead of per-ledger

- **Status:** Accepted

## Context
Payroll is agreed in wall-clock terms (salary per month). Ledger close time is about 5 s but varies.

## Decision
Accrual is `elapsed_seconds × rate_per_second`, using `env.ledger().timestamp()`.

## Alternatives considered
- **Per-ledger** (`sequence()`): pay would drift with ledger close time, so the same stream would pay different amounts per real-world month. Rates would be hard for employers to reason about.

## Consequences
- Rates map directly to salaries (monthly ÷ 2 592 000).
- Granularity is effectively one ledger, since the timestamp only advances when a ledger closes.
- Durations (`stop_time`, cliffs, `TIMELOCK_DELAY`, grace periods) are all in seconds. Ledger counts are used only for TTLs and the per-ledger rate limit.

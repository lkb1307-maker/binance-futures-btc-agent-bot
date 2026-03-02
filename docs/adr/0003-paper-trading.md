# ADR 0003 — Paper trading first as primary verification path

## Status

Accepted

## Context

We must verify logic safely and deterministically before integration complexity.

## Decision

- Implement a Paper Simulator early.
- Paper sim must support:
  - fees
  - slippage (simple model acceptable)
  - partial fills (simplified acceptable)
- E2E tests rely on paper mode to validate invariants.

## Consequences

- Faster and safer iteration loop.
- Some real-world behaviors differ; later validated in Testnet.

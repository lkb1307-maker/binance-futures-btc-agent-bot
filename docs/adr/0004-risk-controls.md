# ADR 0004 — Risk layer as authoritative veto

## Status

Accepted

## Context

“Never-die” requires that risk controls cannot be bypassed by strategy or execution.

## Decision

- Risk module is authoritative: it can veto any trade intent.
- Enforce hard constraints:
  - Max leverage (default <= 2x)
  - Per-trade risk cap (as % of equity)
  - Daily loss limit (stop trading after limit)
  - Consecutive loss cooldown
  - Volatility gating (no trade outside acceptable regime)
- Any order must include a stop-loss at entry time where feasible.

## Consequences

- Fewer trades; higher survival probability.
- Strategy cannot “force” trades; it must operate within constraints.

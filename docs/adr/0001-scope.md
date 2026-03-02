# ADR 0001 — Scope: BTC-only survival-first bot

## Status

Accepted

## Context

We want a robust, low-risk automated trading system. Complexity increases failure modes.

## Decision

- Trade only BTCUSDT perpetual.
- Prioritize survival and risk controls over profit maximization.
- No production trading by default.

## Consequences

- Faster iteration and clearer risk modeling.
- Less diversification; strategy must adapt via volatility gating and strict limits.
- Future expansion to other symbols requires new ADR and explicit acceptance criteria.

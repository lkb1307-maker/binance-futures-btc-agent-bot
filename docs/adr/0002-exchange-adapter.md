# ADR 0002 — Exchange Adapter isolation

## Status

Accepted

## Context

We need an architecture where strategy/risk code is testable without real exchange connectivity.

## Decision

- All exchange calls must be isolated under src/exchange/.
- Strategy and risk layers must not import or depend on exchange client implementation.
- Exchange adapter exposes a narrow interface:
  - get_market_data()
  - get_account_state()
  - place_order()
  - cancel_order()
  - get_open_orders()
  - reconcile()

## Consequences

- Paper simulator and Testnet share the same strategy/risk/execution logic.
- Lower coupling, easier testing, fewer accidental production calls.

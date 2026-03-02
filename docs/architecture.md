# Architecture — Binance Futures BTC Never-Die Bot

## Goals

- Build a survival-first automated trading bot for Binance Futures BTCUSDT perpetual.
- Verify behavior in Paper + Binance Futures Testnet before any production consideration.
- Make the strategy exchange-agnostic and enforce risk controls as hard constraints.
- Enable autonomous development workflow: branch → PR → CI → merge.

## Non-Goals

- Multi-asset trading (BTC only).
- High leverage or aggressive strategies.
- Any mode that can place real-money trades without explicit approval.

## System Overview

This system is composed of 6 layers:

1) Market Data Layer

- Pulls market data (candles, mark price, funding, etc.).
- Normalizes into internal models used by strategy.

2) Strategy Layer (exchange-agnostic)

- Generates intents (LONG/SHORT/FLAT) and target risk parameters.
- Must NOT place orders or call exchange APIs directly.

3) Risk Layer (authoritative veto)

- Applies position sizing, daily loss limits, cooldowns, volatility filters, max leverage limits.
- Can veto any trade. If risk vetoes, no order is allowed.

4) Execution Layer

- Converts approved trade intents into orders.
- Enforces idempotency, retries, and reconciliation on startup.
- Must attach stop-loss at entry time (where possible).

5) Exchange Adapter Layer

- Encapsulates Binance Futures Testnet API interactions.
- Is the only layer allowed to talk to external exchange endpoints.

6) Persistence & Observability Layer

- Structured logs (JSON) + sqlite persistence for decisions, orders, fills, errors.
- Enables post-mortem and reproducibility.

## Data Flow

Market Data → Strategy Signal → Risk Check/Veto → Execution Plan → Order Router → Exchange Adapter → Fill/State Update → Persistence/Logs

### Event Types (recommended)

- DECISION: strategy decision with features snapshot
- RISK_BLOCK: risk veto and reason
- ORDER: order request and final payload
- FILL: fills and resulting position state
- ERROR: exceptions, retries, timeouts
- RECONCILE: startup/account reconciliation events

## Runtime Modes

### Paper Mode

- Uses Paper Simulator for fills.
- Used for fast iteration and deterministic testing (seeded randomness).
- Must support slippage, fees, partial fills (simplified acceptable initially).

### Testnet Mode

- Uses Binance Futures Testnet via Exchange Adapter.
- Used to validate integration behavior, idempotency, and resilience.
- Must reconcile positions/orders on startup.

### Production Mode (Not enabled by default)

- Explicitly out-of-scope until acceptance criteria are met and Alvin approves.

## Resilience & Safety

- Fail-closed: on API instability or uncertain state, disable trading.
- Hard daily loss cap: once hit, stop new trades for the day.
- Cooldown after N consecutive losses.
- Startup reconciliation: if existing open positions, sync state and avoid double-ordering.
- Kill switch: environment variable or local flag file to stop new orders immediately.

## Configuration

- Use config file(s) + environment variables.
- Secrets only via environment variables (never commit).
- `.env.example` documents required variables; `.env` is local only.

## Testing Strategy

- Unit tests:
  - Risk limits (daily loss, cooldown)
  - Position sizing correctness
  - Strategy intent generation (deterministic cases)
- E2E tests:
  - Paper run: simulate N steps and ensure invariants (no trade without SL, daily cap stops trading)
- Integration tests (later):
  - Exchange adapter request signing/serialization (without real network calls where possible)

## Acceptance Criteria (Demo)

Before any production discussion:

- 7-day continuous Testnet run without crashes.
- Verified: daily loss limit triggers and stops trading.
- Verified: network/API errors do not cause order spam.
- Verified: restart reconciliation prevents duplicate orders.
- All critical events are traceable from logs + sqlite.

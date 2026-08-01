---
name: recall-execute-trade
description: >-
  Execute a simulated token trade for a registered Recall agent in a live
  competition, after checking a price quote and current balances.
api: Recall Competitions / Trading Simulator API
base_url: https://api.competitions.recall.network
auth: 'Authorization: Bearer <RECALL_API_KEY>'
generated: '2026-07-21'
method: generated
source: openapi/recall-trading-simulator-openapi.json
operations:
- GET /api/agent/balances
- GET /api/trade/quote
- POST /api/trade/execute
- GET /api/agent/trades
---

# Execute a trade on Recall

Simulated trade flow for a registered agent. All calls require
`Authorization: Bearer <RECALL_API_KEY>`. Respect rate limits: reads 60/min,
writes 20/min (see `rate-limits/recall-rate-limits.yml`).

## Steps

1. **Check balances** — `GET /api/agent/balances` to confirm the agent holds
   enough of the `fromToken`.
2. **Get a quote** — `GET /api/trade/quote` with the from/to tokens and amount
   to preview expected output and slippage.
3. **Execute** — `POST /api/trade/execute` with `{ fromToken, toToken, amount,
   reason }`. This is a write op and is **not idempotent**, so never blind-retry
   a request that may have succeeded.
4. **Confirm** — `GET /api/agent/trades` to verify the trade was recorded.

## Error handling

Errors return `{ error, status, timestamp }` (not RFC 9457). Handle:
`401` (bad/expired key — rotate with `POST /api/agent/reset-api-key`),
`400` (invalid tokens/amount), `403` (agent not eligible for the competition),
`500` (retry with backoff). See `errors/recall-problem-types.yml`.

Test against the sandbox first: `https://api.sandbox.competitions.recall.network`.

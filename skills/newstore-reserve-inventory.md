---
name: Reserve inventory idempotently
description: Check available-to-promise stock and create an inventory reservation using an idempotency key.
api: openapi/newstore-api-openapi-original.json
operations: [listAtpInsights, createReservation, showReservation, listReservations]
---

# Reserve inventory idempotently (NewStore)

## Auth
OAuth 2.0 client-credentials bearer token; requires
`inventory:reservations:write` (and `inventory:reservations:read` to read).
Base URL: `https://{tenant}.p.newstore.net`.

## Steps
1. **Check availability** — `listAtpInsights` (`GET /v0/stock/insights`) to
   confirm available-to-promise quantities for the products/locations.
2. **Reserve** — `createReservation` (`POST /inventory/reservations`). This
   operation **requires an `Idempotency-Key` header**. If a request with the
   same key was already received, NewStore returns the ID of the existing
   reservation instead of creating a duplicate.
3. **Confirm** — `showReservation`
   (`GET /inventory/reservations/{reservation_id}`) to verify status.
4. **List** — `listReservations` (`GET /inventory/reservations`) to reconcile.

## Rules
- Always send a stable `Idempotency-Key` per logical reservation; reuse it on retry.
- Errors are `application/problem+json`; `409`/`423` indicate conflict/locked state.
- `429` means rate-limited — retry with backoff and the same idempotency key.

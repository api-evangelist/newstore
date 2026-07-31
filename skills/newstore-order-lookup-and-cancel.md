---
name: Look up an order and process a cancellation
description: Retrieve a NewStore sales order and customer order history, then cancel the order or an individual line item.
api: openapi/newstore-api-openapi-original.json
operations: [showOrderHistoryForCustomer, showSalesOrderByIdAndIdType, showCancellationStatus, createCancellation, createLineItemCancellation]
---

# Look up an order and process a cancellation (NewStore)

## Auth
OAuth 2.0 client-credentials bearer token. Base URL:
`https://{tenant}.p.newstore.net`.

## Steps
1. **Find the order** — `showOrderHistoryForCustomer` (`GET /v0/orders`) to list
   a customer's orders, or `showSalesOrderByIdAndIdType`
   (`GET /v0/orders/{id}`) to fetch one by id and `id_type`.
2. **Check cancellability** — `showCancellationStatus`
   (`GET /v0/orders/{id}/cancel`) before acting.
3. **Cancel** — either the whole order with `createCancellation`
   (`POST /v0/orders/{id}/cancel`) or a single item with
   `createLineItemCancellation`
   (`POST /v0/orders/{id}/items/{item_uuid}/cancel`).

## Rules
- Errors are `application/problem+json`; a `409` means the order is in a state
  that cannot be cancelled.
- Respect `429` rate limiting with backoff.
- Cancellation behavior is governed by tenant routing config
  (`cancel_on_conflict`, external fulfillment settings).

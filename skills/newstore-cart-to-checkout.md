---
name: Build a cart and identify the customer
description: Create a NewStore checkout cart, add line items, and attach a customer profile ahead of checkout.
api: openapi/newstore-api-openapi-original.json
operations: [createCart, showCart, createLineItem, updateLineItem, createOrShowProfile]
---

# Build a cart and identify the customer (NewStore)

Use the NewStore Omnichannel API to assemble a cart and associate a shopper.

## Auth
Obtain an OAuth 2.0 client-credentials token from the tenant realm
(`https://id.p.newstore.net/auth/realms/{tenant}/protocol/openid-connect/token`)
and send it as `Authorization: Bearer <token>`. Requires the
`checkout:carts:write` and `customer:profile:write` scopes.
Base URL: `https://{tenant}.p.newstore.net`.

## Steps
1. **Create the cart** — `createCart` (`POST /checkout/carts`). Capture the
   returned `cartId`.
2. **Add items** — `createLineItem` (`POST /checkout/carts/{cartId}/items`) for
   each product. Adjust with `updateLineItem`
   (`PATCH /checkout/carts/{cartId}/items/{lineItemId}`) if quantities change.
3. **Identify the customer** — `createOrShowProfile`
   (`POST /customer/profiles`) to create or resolve the shopper's profile.
4. **Re-read state** — `showCart` (`GET /checkout/carts/{cartId}`) to confirm
   totals before handing off to checkout/payment.

## Rules
- Errors come back as `application/problem+json`; read `title`/`detail`/`error_code`.
- Expect `429` under rate limiting — back off and retry.
- Handle `403` as a missing-scope error, not a not-found.

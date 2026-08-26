---
name: Build a MAKEUP BY MARIO cart
description: >-
  Create and populate a cart on the MAKEUP BY MARIO storefront through either MCP server or
  the GraphQL cart mutations, set buyer identity and delivery, and stop at the checkout
  handoff.
api: mcp/makeup-by-mario-mcp.yml
graphql: graphql/makeup-by-mario-storefront.graphql
operations: [create_cart, get_cart, update_cart, cancel_cart, cartCreate, cartLinesAdd, cartLinesUpdate, cartLinesRemove, cartBuyerIdentityUpdate, cartDeliveryAddressesAdd, cartSelectedDeliveryOptionsUpdate, cartDiscountCodesUpdate]
generated: '2026-08-25'
method: generated
---

# Build a MAKEUP BY MARIO cart

You have a variant id from `makeup-by-mario-find-shade.md`. This skill takes it to a cart
ready for a human to check out. **It stops before payment** — see
`makeup-by-mario-complete-checkout.md` for why that boundary is not negotiable here.

## Surfaces

| Surface | Operations |
|---|---|
| UCP MCP `https://www.makeupbymario.com/api/ucp/mcp` | `create_cart`, `get_cart`, `update_cart`, `cancel_cart` |
| Storefront MCP `https://www.makeupbymario.com/api/mcp` | `get_cart`, `update_cart` |
| GraphQL `https://www.makeupbymario.com/api/2026-04/graphql.json` | the `cart*` mutations |

## Steps — MCP path

1. `create_cart` (UCP) with the initial line items. Required inputs are `meta` and `cart`.
2. `update_cart` for everything after that — it is the consolidated write. One MCP tool
   stands in for eight or more GraphQL mutations: line add/update/remove, buyer identity,
   discount codes, delivery addresses, attributes and note.
3. `get_cart` after every mutation and read the authoritative state back. Do not assume your
   write landed as sent.
4. `cancel_cart` (UCP) if the shopper abandons. There is no GraphQL equivalent — this
   reversal exists only on the UCP server.

## Steps — GraphQL path

`cartCreate` → `cartLinesAdd` / `cartLinesUpdate` / `cartLinesRemove` →
`cartBuyerIdentityUpdate` → `cartDeliveryAddressesAdd` →
`cartSelectedDeliveryOptionsUpdate` → `cartDiscountCodesUpdate`.

## Rules

- **Read `userErrors` on every GraphQL mutation.** A mutation can return HTTP 200 with an
  empty top-level `errors[]` and still have failed; the failure is in the payload's
  `cartUserErrors` / `userErrors` field, typed by the `CartErrorCode` enum.
- **JSON-RPC errors come back as HTTP 200.** On both MCP servers, an error is a `200` whose
  body carries an `error` member. Branching on status code alone will read a failure as a
  success.
- **There is no idempotency key on any cart operation.** A retried `create_cart` creates a
  second cart. Track the cart id you got back and reuse it.
- **One shipping destination per checkout.** `/.well-known/ucp` declares
  `allows_multi_destination.shipping: false` and `allows_method_combinations: [["shipping"]]`.
- Do not use `/cart.js` or `/recommendations/products`. The store's own `robots.txt`
  disallows both and says agents should use UCP/MCP instead.

## Reversal

Everything in this skill is reversible. `cancel_cart` undoes the cart; there is no stated
expiry window on a cart, so treat cancellation as your own housekeeping rather than
something the store will do for you.

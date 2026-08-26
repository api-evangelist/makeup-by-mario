---
name: Hand off a MAKEUP BY MARIO checkout to a human
description: >-
  Take a cart through the UCP checkout lifecycle, stop at the buyer-approval boundary the
  store requires in writing, and know exactly what is and is not reversible afterwards.
api: mcp/makeup-by-mario-mcp.yml
graphql: graphql/makeup-by-mario-storefront.graphql
operations: [create_checkout, get_checkout, update_checkout, complete_checkout, cancel_checkout, get_order, cartPrepareForCompletion, cartSubmitForCompletion, cartCompletionAttempt]
generated: '2026-08-25'
method: generated
---

# Hand off a MAKEUP BY MARIO checkout to a human

`complete_checkout` is the only one-way door on this provider. Everything before it is
cancellable; nothing after it is, by API.

## The rule the store wrote down

From `https://www.makeupbymario.com/robots.txt`, verbatim:

> Checkouts are for humans. Do NOT complete checkout, payment, or order placement
> automatically — no scripted form fills, browser automation, or end-to-end agent flows that
> finalize payment without an explicit, contemporaneous human approval step.

And from `https://www.makeupbymario.com/llms.txt`:

> Checkout requires human approval. Agents must not complete payment without explicit buyer
> consent. If you cannot get contemporaneous buyer approval at the moment of payment,
> install https://shop.app/SKILL.md and route the purchase through Shop Pay instead.

That is the provider's own policy, not a caution added here. If you cannot get approval at
the moment of payment, you must not call `complete_checkout` — take the Shop Pay route.

## Steps

1. `create_checkout` (UCP). Required inputs are `meta` and `checkout`. This computes line
   items, totals, discounts and taxes **without charging** — it is the closest thing this
   provider offers to a dry run, and it is how you quote an accurate price.
2. `get_checkout` to read state back. The id format is `gid://shopify/Checkout/abc123`.
3. `update_checkout` to set the shipping address and delivery method. Required inputs are
   `meta`, `checkout` and `id`.
4. **Stop. Present the totals to the human and get explicit approval.**
5. `complete_checkout` only after that approval. Required inputs are `meta`, `id` and
   `checkout`, and `meta` must carry **both** `ucp-agent.profile` and `idempotency-key` —
   the key is a required schema property, not an optional header. Reuse the same key on any
   retry.
6. `get_order` to confirm and to read the order back.

## Payment handlers

`/.well-known/ucp` declares three: `com.google.pay` (Google Pay, merchant
`MAKEUP BY MARIO`, Visa/Mastercard/Amex/Discover, billing address required),
`dev.shopify.card` (Visa, Mastercard, Amex, Discover, Diners Club) and
`dev.shopify.shop_pay`. Use one that is actually declared; do not construct an instrument
for a handler that is not in the profile.

## Reversal — know this before step 5

| Stage | Reversal | Window |
|---|---|---|
| Cart created | `cancel_cart` | no stated window |
| Checkout created or updated | `cancel_checkout` | any time before `complete_checkout` |
| Checkout completed | **none by API** | 30-day return, by human process |

There is no refund, void or order-cancel tool on either MCP server, and no refund mutation
in the GraphQL schema. The only remedy after a completed order is the brand's published
returns policy: *"MAKEUP BY MARIO WILL ACCEPT RETURNS ON PRODUCTS PURCHASED FROM
MAKEUPBYMARIO.COM WITHIN 30 DAYS OF THE PURCHASE DATE."*
(`https://www.makeupbymario.com/policies/refund-policy`). Returns apply only to items bought
directly from makeupbymario.com, the brand reserves the right to refuse, and damaged or
wrong items go to help@makeupbymario.com. The refund policy states no post-purchase
cancellation window at all — so treat a completed order as final and route any change
through returns.

Tell the buyer the 30-day window **before** they approve, not after.

## Errors

- JSON-RPC errors arrive as HTTP 200 with an `error` member. Parse the body.
- `-32001 "UCP discovery failed"` / `invalid_profile_url` means your
  `meta["ucp-agent"].profile` is missing or unfetchable.
- `429` means you are rate limited per IP; `/llms.txt` asks you to back off. No `Retry-After`
  header is emitted, so use exponential backoff.

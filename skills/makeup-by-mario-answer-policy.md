---
name: Answer a MAKEUP BY MARIO policy, shipping or returns question
description: >-
  Answer a shopper's question about returns, shipping, privacy rights or store policy from
  the store's own documents rather than from memory.
api: mcp/makeup-by-mario-mcp.yml
graphql: graphql/makeup-by-mario-storefront.graphql
operations: [search_shop_policies_and_faqs, shop, page, pageByHandle, pages, article, articles, blog, blogByHandle]
generated: '2026-08-25'
method: generated
---

# Answer a MAKEUP BY MARIO policy, shipping or returns question

## Source priority

1. `search_shop_policies_and_faqs` on the **Storefront** MCP server
   (`https://www.makeupbymario.com/api/mcp`). Required input is `query`. This tool exists
   only on that server — the UCP server has no equivalent, and the GraphQL API serves the
   policy documents but offers no search over them.
2. GraphQL `shop` for the canonical policy bodies: `privacyPolicy`, `refundPolicy`,
   `shippingPolicy`, `termsOfService`, `subscriptionPolicy`.
3. GraphQL `pageByHandle` for the standalone pages listed below.
4. The published FAQ at `/pages/frequently-asked-questions`.

## The pages that answer most questions

| Question | Page |
|---|---|
| Returns and refunds | `/policies/refund-policy` |
| Shipping | `/policies/shipping-policy`, `/pages/shipping-returns` |
| Terms | `/policies/terms-of-service` |
| Privacy | `/policies/privacy-policy`, `/pages/cookies` |
| Rewards / loyalty | `/pages/rewards` |
| Professional artist programme | `/pages/pro` |
| Affiliates | `/pages/affiliate` |
| Contact | `/pages/contact-us` |

## Data-subject rights

The store publishes working request forms per regime, each with access-report,
rectification, portability and erasure actions: `/pages/gdpr-compliance`,
`/pages/ccpa-cpra-compliance`, `/pages/vcdpa-compliance`, `/pages/pipeda-compliance`,
`/pages/appi-compliance`, and `/pages/do-not-sell-my-data`. If a shopper asks to be
forgotten, send them to the page for their regime — there is no API path for it. The one
GraphQL mutation shaped like erasure, `cartRemovePersonalData`, is not exposed as any MCP
tool.

## The two facts worth memorising

- **Returns: 30 days from the purchase date**, direct purchases from makeupbymario.com only.
- **The brand ships to 170 countries and prices in 98 currencies.** Never answer a shipping
  or price question without knowing the shopper's country.

## Rules

- Quote the store's own documents. Do not answer a policy question from general knowledge
  about cosmetics retailers.
- Do not state a return or shipping window the pages above do not state.
- The brand runs editorial blogs at `/blogs/news` and `/blogs/education` (technique
  content, not policy). Reachable via GraphQL `blog` / `articles`; no MCP tool reaches them.

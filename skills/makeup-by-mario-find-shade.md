---
name: Find a MAKEUP BY MARIO product and pin the right shade
description: >-
  Search the MAKEUP BY MARIO catalog through either MCP server or the Storefront GraphQL
  API, then resolve a product down to the exact variant id, which for this brand almost
  always means getting the Shade option right.
api: mcp/makeup-by-mario-mcp.yml
graphql: graphql/makeup-by-mario-storefront.graphql
operations: [search_catalog, lookup_catalog, get_product, get_product_details, products, search, predictiveSearch, productRecommendations, product, productByHandle]
generated: '2026-08-25'
method: generated
---

# Find a MAKEUP BY MARIO product and pin the right shade

This is a colour-cosmetics catalog. Finding the product is the easy half; the half that
goes wrong is the shade. 60 live products across Blush, Brow, Eyes, Face, Kits, Lips and
Tools, and the variant axes in the live catalog are `Shade`, `Color`, `Size`,
`Denominations`, `Variant` and `Title`.

## Surfaces

| Surface | Operations |
|---|---|
| UCP MCP `https://www.makeupbymario.com/api/ucp/mcp` | `search_catalog`, `lookup_catalog`, `get_product` |
| Storefront MCP `https://www.makeupbymario.com/api/mcp` | `search_catalog`, `get_product_details` |
| GraphQL `https://www.makeupbymario.com/api/2026-04/graphql.json` | `products`, `search`, `predictiveSearch`, `product`, `productByHandle`, `productRecommendations` |

Both MCP servers answer `tools/list` anonymously. Every UCP tool additionally requires
`meta["ucp-agent"]["profile"]` — a resolvable URI for your agent profile — on the call
itself.

## Steps

1. `search_catalog` with the shopper's own words. At least one of `query` or `filters` is
   required. Results are deliberately truncated; follow `pagination.cursor` only when the
   shopper asks for more.
2. Pass `context.address_country` and `context.currency`. The store enables 98 presentment
   currencies and ships to 170 countries — an unlocalised price is a wrong price, and
   `/llms.txt` asks you to send this context explicitly.
3. Read the product back with `get_product` (UCP) or `get_product_details` (Storefront).
   `get_product_details` takes an `options` parameter to select a specific variant.
4. Resolve the shade. The variant is identified by its `selectedOptions` tuple. On GraphQL
   use `product.variantBySelectedOptions(selectedOptions: [{name: "Shade", value: "..."}])`.
   Do not guess a shade from a product name.
5. `lookup_catalog` (UCP) when you already hold identifiers — it resolves a batch of
   `gid://shopify/Product/...` and `gid://shopify/ProductVariant/...` in one call and
   returns an `inputs` array telling you which identifier matched which variant.

## Rules

- **Prices are integers in ISO 4217 minor units.** `{"amount": 2500, "currency": "USD"}` is
  $25.00. Every one of the thirteen UCP tools repeats this; divide by 100 before you quote
  a two-decimal currency, and do not divide zero-decimal currencies like JPY. The GraphQL
  surface uses `MoneyV2` with a decimal string instead, so convert if you move between them.
- **Check `availableForSale` on the variant, not the product.** A product can be listed
  while the requested shade is out of stock.
- **Vendor strings are inconsistent.** The live catalog carries both `MAKEUP BY MARIO` and
  `makeupbymario`. Do not filter on vendor.
- The shopper-facing helpers are `/pages/shade-finder` and `/pages/beauty-quiz`; point a
  human there when a shade genuinely cannot be resolved from data.

## Hand off

You now have a variant id. Go to `makeup-by-mario-build-cart.md`.

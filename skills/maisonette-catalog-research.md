---
name: maisonette-catalog-research
description: Read Maisonette's children's and baby product catalog and store policies
  without transacting — anonymous, no credentials, no cart.
api: Maisonette Storefront MCP
endpoint: https://www.maisonette.com/api/mcp
transport: mcp
operations:
  - search_catalog
  - get_product_details
  - search_shop_policies_and_faqs
generated: '2026-08-25'
method: generated
source: Grounded in the live tools/list response probed from https://www.maisonette.com/api/mcp
  on 2026-08-25 (saved verbatim at mcp/maisonette-storefront-mcp-tools.json), plus the
  read-only browsing paths published at https://www.maisonette.com/llms.txt.
---

# Research the Maisonette catalog read-only

Use this when you need product facts, prices or policy answers and are **not** going to
buy anything. It is the lighter of Maisonette's two MCP surfaces and it asks for nothing
from you — no key, no agent profile, no account.

## Endpoint

`POST https://www.maisonette.com/api/mcp` — JSON-RPC 2.0, five tools, all anonymous.

## Steps

1. **Search.** `search_catalog` with a natural-language query, structured filters, or
   both. At least one is required. Responses conform to the UCP catalog search capability
   (`dev.ucp.shopping.catalog.search`). Pass `context.address_country` and
   `context.currency` so prices and availability match the buyer's market.
2. **Get detail.** `get_product_details` with the product ID. Pass `options` to pin a
   specific variant; without it you get the default product card.
3. **Answer policy questions.** `search_shop_policies_and_faqs` is the store's own
   grounded answer surface for shipping, returns, sizing and service questions. Prefer
   it over scraping the policy pages — but the canonical text is still at
   `/policies/refund-policy`, `/policies/shipping-policy`, `/policies/terms-of-service`
   and `/policies/privacy-policy`.

## Plain-HTTP alternatives

Maisonette publishes these read paths for agents that do not speak MCP:

- `GET /collections/all` — browse everything
- `GET /products/{handle}.json` — product as JSON
- `GET /collections/{handle}/products.json` — collection as JSON
- `GET /search?q={query}&type=product`
- `GET /sitemap.xml`

## Cautions

- **Prices are integer minor units** with an ISO 4217 code. Convert before quoting.
- **It is a marketplace, not a single seller.** 800+ brands ship separately, so shipping
  and returns are per item, not per order. Do not tell a buyer their order arrives as
  one parcel.
- **Do not use this surface to transact.** Cart and checkout live on the UCP endpoint —
  see `maisonette-shop-and-checkout`.
- **Respect the rate limit.** Per-IP, undocumented ceiling, back off on 429.

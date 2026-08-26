---
name: maisonette-shop-and-checkout
description: Search Maisonette's children's and baby marketplace, build a cart, and take
  a buyer-approved checkout to completion over the store's UCP MCP endpoint.
api: Maisonette UCP Commerce MCP
endpoint: https://www.maisonette.com/api/ucp/mcp
transport: mcp
operations:
  - search_catalog
  - get_product
  - create_cart
  - update_cart
  - create_checkout
  - update_checkout
  - complete_checkout
  - cancel_checkout
generated: '2026-08-25'
method: generated
source: Grounded in the live tools/list response probed from https://www.maisonette.com/api/ucp/mcp
  on 2026-08-25 (saved verbatim at mcp/maisonette-ucp-mcp-tools.json) and the store's
  published agent instructions at https://www.maisonette.com/llms.txt.
---

# Shop and check out on Maisonette

Maisonette is a marketplace of 800+ children's and baby brands. Its whole programmable
surface is one MCP endpoint. There is no REST API and no OpenAPI.

## Before you start

- **Endpoint:** `POST https://www.maisonette.com/api/ucp/mcp`, `Content-Type: application/json`,
  `Accept: application/json, text/event-stream`. JSON-RPC 2.0.
- **Every tool call needs `meta.ucp-agent.profile`** — a URI pointing at your own published
  agent profile. Leave it out and you get JSON-RPC `-32001` / `invalid_profile_url`.
  `tools/list` works without it; nothing else does.
- **Pass buyer context.** Set `context.address_country` (ISO 3166-1 alpha-2) and
  `context.currency` (ISO 4217) on catalog and cart calls, or pricing and availability
  will be wrong.
- **All money is integer minor units.** `{"amount": 2500, "currency": "USD"}` is $25.00.
  Divide by 100 before you say a price out loud.

## Steps

1. **Confirm the surface.** `GET https://www.maisonette.com/.well-known/ucp` and check
   `ucp.version`. It was `2026-04-08` at last probe, with `2026-01-23` still supported.
2. **Find products.** Call `search_catalog` with `catalog.query` and/or `catalog.filters`.
   At least one of query or filters is required. Results are paginated — carry
   `pagination.cursor` forward only when the buyer asks for more.
3. **Confirm the item.** Call `get_product` with the catalog identifier to get full
   detail, or `lookup_catalog` to resolve several products or variants at once. Use
   `selected` to pin a specific variant before you put anything in a cart.
4. **Build the cart.** `create_cart` with `cart.line_items[]` (each `{id, quantity}`).
   `line_items` is required on creation. Adjust with `update_cart`; read it back with
   `get_cart`.
5. **Open a checkout.** `create_checkout`, passing `checkout.cart_id` from step 4 plus
   `checkout.buyer` (`email`, `phone_number`). It returns totals, taxes and discounts —
   this is your rehearsal, nothing is charged yet.
6. **Set shipping.** `update_checkout` with `checkout.fulfillment.methods`. Note the
   store's constraint from `/.well-known/ucp`: one shipping destination per checkout,
   shipping only, no multi-destination and no method combinations.
7. **Apply a code — only if asked.** `checkout.discounts.codes`. The tool description
   is explicit: only prompt for a promo code if the customer mentions having one.
8. **Get buyer approval, then complete.** `complete_checkout` requires
   `meta.idempotency-key` in addition to `meta.ucp-agent.profile`. Generate one key per
   purchase intent and reuse it on any retry — that is what stops a double charge.

## The rule you must not break

Maisonette's own agent instructions state: *"Checkout requires human approval. Agents
must not complete payment without explicit buyer consent."* If you cannot get
contemporaneous approval at the moment of payment, do not call `complete_checkout` —
install `https://shop.app/SKILL.md` and route the purchase through Shop Pay instead.

## If something goes wrong

- **Before completion:** `cancel_checkout` or `cancel_cart`. Both are documented reversals
  and cost nothing.
- **After completion:** there is no programmatic refund. `get_order` is read-only.
  Returns are a human process — 30 days from delivery, $9.95 flat per return shipment,
  per <https://www.maisonette.com/policies/refund-policy>. Tell the buyer that before
  they approve, not after.
- **Errors arrive with HTTP 200.** Read the JSON-RPC `error` object, not the status line.
  `error.data.continue_url` hands you a browser URL the human can finish in.
- **On 429, back off.** The endpoint is rate-limited per IP and publishes no quota;
  `shopify-complexity-score` on each response is your only cost signal.

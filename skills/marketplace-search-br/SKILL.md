---
name: marketplace-search-br
description: Gets data from Shopee Brazil without login: today's and tomorrow's flash sales with units sold and stock, shop reputation and featured product sales, shop reviews by star, product details with stock per variation, keyword suggestions and official shops. Use when the user asks what sells on Shopee Brazil, wants to check a seller or product link, find bad reviews of a shop, or research keywords buyers use. Runs on Apify with the user's own APIFY_TOKEN; pay per product, shop or review delivered.
license: MIT
---

# Marketplace search (Shopee Brazil)

One Apify Actor by Pom Labs reads public data from Shopee Brazil (shopee.com.br) and returns it as JSON. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~shopee-brazil-scraper` |
| Price | $0.0003 per product delivered ($0.3 per 1,000), $0.003 per shop delivered ($3 per 1,000), $0.0015 per review delivered ($1.5 per 1,000) and $0.0003 per keyword suggestion ($0.3 per 1,000); errors are free |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | Seconds for tens of rows |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/shopee-brazil-scraper if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/shopee-brazil-scraper`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get Shopee data

```bash
# mode: auto, flash_sales, discover, shop, shop_reviews, product, keywords or official_shops
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~shopee-brazil-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"mode":"flash_sales","maxResults":20}'
```

The response is a JSON array: one object per product, shop, review or suggestion, marked by `type`.

## Input fields

| Field | Meaning |
| --- | --- |
| `mode` | `auto` (default: a product link gets the product, a shop link or id the shop, a word the flash sales filtered by it), `flash_sales`, `discover`, `shop`, `shop_reviews`, `product`, `keywords`, `official_shops` |
| `queries` | Shop links, usernames or ids; product links; or keywords, depending on the mode. Up to 50 |
| `maxResults` | Rows per query, default 50 (reviews up to 2,000 per shop). This is the cost cap: always set it |
| `session` | For flash sales: `all` (default), `current` or `next` |
| `reviewStars` | For `shop_reviews`: 1 to 5 for only those stars; 0 = all |
| `includeCouponsAndCategories` | For `shop`: also the coupons and categories (default on) |
| `minPrice`, `maxPrice`, `minRating` | Filters applied to flash sale and discover rows |

There are more fields on the Actor's page (https://apify.com/pomlabs/shopee-brazil-scraper); these are the main ones.

## Output fields worth keeping

Products: `name`, `url`, `price`, `sold`, `stock`, `rating`, `ratingCount`, `saleEnd`. Shops: `username`, `followers`, `ratingsGood`, `ratingsNormal`, `ratingsBad`, `responseRate`, `cancellationRate`, `officialShop`, `itemCount`, `featuredProduct`, `coupons`. Reviews: stars, text, product, photos and the seller reply.

Prices are in Brazilian reais. Only Shopee Brazil is read.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Empty result for a shop or product | The link is wrong or the item was removed | Check the link; errors are free |

## Cost check before big batches

Cost is `queries x maxResults x` the event price (products $0.0003, reviews $0.0015, shops $0.003) at most. Above $5, tell the user the estimate and wait for a yes.

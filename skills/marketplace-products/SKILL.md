---
name: marketplace-products
description: Gets AliExpress search results as JSON - product title, price, original price and discount, units sold, rating, store, shipping and delivery dates - for a keyword or a search page, as a shopper in a chosen country sees them. Use when the user wants AliExpress prices for a product, wants to compare suppliers or find best sellers for dropshipping or product research, wants to track the price of an item across sellers, or shares an AliExpress search link. Runs on Apify with the user's own APIFY_TOKEN; pay per product, no AliExpress login.
license: MIT
---

# Marketplace products (AliExpress)

One Apify Actor by Pom Labs returns the products of AliExpress search pages. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~aliexpress-scraper` |
| Price | $0.001 per product ($1 per 1,000) |
| Auth | `Authorization: Bearer $APIFY_TOKEN`. No AliExpress account is used |
| Typical time | 5 to 15 s for the first 60 products |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/aliexpress-scraper if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to `pomlabs/aliexpress-scraper` and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get products

`maxItems` is the cost cap: always set it. AliExpress shows 60 products per page.

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~aliexpress-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"keywords":["wireless earbuds"],"country":"US","sort":"orders","maxItemsPerSearch":60,"maxItems":60}'
```

The response is a JSON array with one object per product, in the order of the site.

## Input fields

| Field | Meaning |
| --- | --- |
| `keywords` | What a shopper would type in the search box. Each keyword is one search |
| `searchUrls` | Search or category result pages copied from the site, with any filters already set |
| `country` | Two-letter code, default `US`. Prices, currency, products and delivery dates follow this country. Ask the user where they buy from when it is not clear |
| `currency` | Three-letter code. Empty uses the country's own currency. Outside the US only |
| `sort` | `relevance` (default), `orders`, `price_asc` or `price_desc` |
| `maxItemsPerSearch` | Default 60. The Actor turns the pages until it reaches this number or the results end |
| `maxItems` | Stop after this many products across all searches (0 = no limit) |

## Output fields worth keeping

`title`, `url`, `productId`, `price`, `originalPrice`, `discountPercent`, `currency`, `sold` and `soldText` (as the site shows it, e.g. `100K+ sold`), `rating`, `storeName`, `freeShipping`, `shippingText`, `deliveryDaysMin`, `deliveryDaysMax`, `isChoice`, `isAd`, `position`, `searchKeyword`, `searchTotalResults`.

- `sold` is rounded by the site (`100K+`), so treat it as a floor.
- `isAd: true` marks a sponsored result: leave it out of "best seller" rankings or flag it.
- Prices are the search-page prices for the chosen country; coupons and shipping can change the final price at checkout.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Empty array | The search has no results in that country | Try another wording or country; say what was searched |
| Item with `status` other than `ok` | AliExpress showed a verification page for that search | Retry that search once, then report it |

## Cost check before big batches

Cost is at most `maxItems x $0.001`. Above 5,000 products ($5), tell the user the estimate and wait for a yes.

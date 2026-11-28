---
name: sold-listings
description: Gets eBay sold (completed) listings by keyword from 13 eBay sites: real sold price, sold date, condition, shipping, auction or buy-it-now, seller and link, plus a free price summary per keyword (recommended price, median, quartiles, sales per day, sell-through rate). Use when the user asks what something actually sells for on eBay, wants resale value, price trends or comparables for a product. Runs on Apify with the user's own APIFY_TOKEN; pay per sold listing.
license: MIT
---

# Sold listings (eBay)

One Apify Actor by Pom Labs reads what items actually sold for on eBay and returns the sold listings and a price summary per keyword. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~ebay-sold-listings` |
| Price | $0.002 per sold listing delivered ($2 per 1,000); $0.004 per item page details delivered ($4 per 1,000) (only with `includeDetails`); the price summary row, empty keywords and blocked pages are free |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | Seconds to a minute for 50 listings; eBay allows about 12 item pages a minute for `includeDetails` |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/ebay-sold-listings if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/ebay-sold-listings`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get sold listings

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~ebay-sold-listings/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"keywords":["nintendo switch oled"],"site":"com","maxItems":50,"soldWithinDays":30,"includeSummary":true}'
```

The response is a JSON array: one row per sold listing, plus one free summary row per keyword.

## Input fields

| Field | Meaning |
| --- | --- |
| `keywords` | What to search, one per line. Each is searched with the Sold filter on |
| `site` | `com`, `co.uk`, `de` (default), `fr`, `it`, `es`, `ca`, `com.au`, `at`, `ch`, `ie`, `nl`, `pl` |
| `maxItems` | Listings per keyword, 1 to 2,000, default 50. This is the cost cap: always set it |
| `soldWithinDays` | Only sales of the last N days |
| `condition` | `new`, `open_box`, `refurbished`, `used`, `for_parts` |
| `minPrice`, `maxPrice` | Price range in the site currency |
| `buyingFormat` | `all`, `auction`, `buyItNow`, `acceptsOffers` |
| `excludeKeywords` | Words to leave out (case, box, broken) |
| `skipOtherModels` | Drop listings that name another model than the keyword |
| `includeSummary` | The free price summary row per keyword (default on) |
| `deltaMode` | Only sales not delivered before for the same keyword, site and filters |
| `includeDetails` | Also open the item page of the first listings: category, item specifics, product ids. Extra charge |

There are more fields on the Actor's page (https://apify.com/pomlabs/ebay-sold-listings); these are the main ones.

## Output fields worth keeping

Listing rows: `title`, sold price and currency, `soldDate`, condition, buying format, bids, shipping, item location, seller name and feedback, `itemId`, link and photo. Summary row: `recommendedPrice`, `priceRange`, median, quartiles, `salesPerDay`, `priceTrendPercent`, sell-through rate and a `confidence` note.

The prices are in the currency of the chosen site. When giving a resale value, quote the summary row and say how many listings it is based on.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Keyword returns no rows | Nothing sold in the period, or the filters are too tight | Widen `soldWithinDays`, drop filters; empty keywords are free |
| A site is reported as unavailable or blocked | eBay refused the page | Retry once later; these rows are free |

## Cost check before big batches

Cost is `keywords x maxItems x $0.002` at most, plus $0.004 per item page when `includeDetails` is on. Above $5, tell the user the estimate and wait for a yes.

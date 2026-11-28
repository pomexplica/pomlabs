---
name: product-reviews
description: Gets Amazon customer reviews with the full text from 22 Amazon sites (Amazon.com.br returns the full list with filters), plus Amazon's own AI "Customers say" summary and aspects, as JSON. Use when the user gives an ASIN or an Amazon product link and wants what buyers love and hate, critical reviews, reviews by star, keyword or date, quality trends over time, or review text for analysis. Runs on Apify with the user's own APIFY_TOKEN; pay per review delivered.
license: MIT
---

# Product reviews (Amazon)

One Apify Actor by Pom Labs returns the reviews of Amazon products, one row per review, from 22 Amazon sites. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~amazon-reviews-deep` |
| Price | $0.0015 per review delivered ($1.5 per 1,000); product and summary rows, wrong ASINs and blocked requests are free |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | Seconds for a few dozen reviews; larger pulls take longer |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/amazon-reviews-deep if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/amazon-reviews-deep`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get reviews

```bash
# ASINs or product URLs of any Amazon site; country "br" is the default for bare ASINs
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~amazon-reviews-deep/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"products":["B09B8XVSDP"],"country":"br","maxReviewsPerProduct":20,"stars":["critical"],"includeProductDetails":true}'
```

The response is a JSON array: one object per review, plus free rows with the product details and the per-product summary.

## Input fields

| Field | Meaning |
| --- | --- |
| `products` (required) | ASINs or product or review page URLs of any Amazon site |
| `country` | Site for bare ASINs: `br` (default), `com`, `uk`, `de`, `fr`, `it`, `es` and others |
| `maxReviewsPerProduct` | 1 to 1,000, default 20. This is the cost cap: always set it. `0` returns only the free product row |
| `stars` | `["1","2"]`, or `["critical"]` (1 to 3) or `["positive"]` (4 to 5) |
| `sort` | `helpful` (default) or `recent` |
| `keywords` | Only reviews that mention one of these words |
| `reviewsNewerThan`, `reviewsOlderThan` | A date (`2026-11-01`) or a period (`30 days`, `6 months`) |
| `verifiedOnly`, `mediaOnly` | Amazon filters for verified purchases and reviews with photos or video |
| `includeProductDetails` | Free row with title, price, star distribution and the AI summary with aspects |
| `includeSummary` | Free row per product with the average by variant and by month (default on) |
| `includeReviewerData` | Set `false` to drop reviewer names and profile links |

There are more fields on the Actor's page (https://apify.com/pomlabs/amazon-reviews-deep); these are the main ones.

## Output fields worth keeping

Review rows: `rating`, `title`, `text`, `date`, `country`, `language`, `verifiedPurchase`, `helpfulVotes`, `variant`, `images`, `reviewUrl`, `asin`, `marketplace`. Product row: `customersSay` and `aspects` (what customers mention, positive and negative counts). Summary row: average rating and share of 1 and 2 star reviews by variant and month.

On Amazon.com.br the Actor reads the full review list with every filter. On the other sites Amazon shows only the reviews on the product page, so say how many were returned and do not present them as the whole picture. Quote reviews exactly.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Few reviews on a non-Brazil site | Amazon shows only a few reviews to visitors who are not signed in there | Say so; use `country: br` or the product row with the AI summary |
| Empty result for an ASIN | Wrong ASIN, or the product has no reviews | Check the ASIN; these rows are free |

## Cost check before big batches

Cost is `products x maxReviewsPerProduct x $0.0015` at most. Above $5 (about 3,300 reviews), tell the user the estimate and wait for a yes.

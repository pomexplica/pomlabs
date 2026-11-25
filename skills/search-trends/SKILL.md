---
name: search-trends
description: Gets Google Trends data as JSON for any keyword - interest over time, peak date and trend change, interest by country or region, and the top and rising related searches - for any country, time range, category and for web, YouTube, News, Images or Shopping search. Use when the user asks whether interest in a topic, product or brand is growing, wants to compare seasons or countries, or wants related and breakout searches for content or product ideas. Runs on Apify with the user's own APIFY_TOKEN; pay per keyword.
license: MIT
---

# Search trends (Google Trends)

One Apify Actor by Pom Labs reads Google Trends for a list of keywords. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~google-trends-scraper` |
| Price | $0.002 per keyword delivered ($2 per 1,000); failures are free |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | 5 to 20 s for a few keywords (median 8 s in our runs) |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/google-trends-scraper if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/google-trends-scraper`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get trends

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~google-trends-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"searchTerms":["air fryer","electric kettle"],"geo":"US","timeRange":"today 12-m","searchType":"web"}'
```

The response is a JSON array with one object per keyword.

## Input fields

| Field | Meaning |
| --- | --- |
| `searchTerms` | Keywords; each one is one result |
| `trendsUrls` | Links copied from trends.google.com/trends/explore; their `q=` keywords are added |
| `geo` | Two-letter country (`US`, `BR`, `GB`) or region (`US-CA`, `BR-SP`); empty = Worldwide |
| `timeRange` | `now 1-H`, `now 4-H`, `now 1-d`, `now 7-d`, `today 1-m`, `today 3-m`, `today 12-m` (default) and longer ranges listed on the Actor page |
| `customTimeRange` | `YYYY-MM-DD YYYY-MM-DD`, overrides `timeRange` |
| `searchType` | `web` (default), `youtube`, `news`, `images`, `shopping` |
| `category` | Google Trends category number (0 = all) |
| `includeRelatedQueries`, `includeInterestByRegion` | Both on by default |
| `language`, `timezoneOffset` | Language of names and dates (`en-US`, `pt-BR`); minutes from UTC |

## Output fields worth keeping

`searchTerm`, `geo`, `timeRange`, `interestOverTime` (list of `date` and `value`, 0 to 100), `average`, `peakDate`, `trendChangePercent`, `interestByRegion`, `relatedQueriesTop`, `relatedQueriesRising` (with "Breakout" values), `trendsUrl`, `hasData`.

Google Trends values are relative (100 = the peak in that period and place), not search counts. Keywords from separate results are not on the same scale; say so when comparing them.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| `hasData: false` | Too little search volume for that keyword, place and period | Widen the period or the place |

## Cost check before big batches

Cost is `keywords x $0.002`. Above 2,000 keywords ($4), tell the user the estimate and wait for a yes.

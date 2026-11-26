---
name: search-trends
description: Gets Google Trends data as JSON for any keyword - interest over time, peak date and trend change, interest by country, region or US metro area, the top and rising related searches, and comparisons of up to 5 keywords on one scale with the leader - plus the Trending now list of any country (what people search right now, with volume, % increase and, when Google has them, news). Use when the user asks whether interest in a topic, product or brand is growing, wants to compare keywords, seasons or countries, wants related and breakout searches for content or product ideas, or asks what is trending today in a country. Runs on Apify with the user's own APIFY_TOKEN; pay per keyword or per trending topic.
license: MIT
---

# Search trends (Google Trends)

One Apify Actor by Pom Labs reads Google Trends for a list of keywords. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~google-trends-scraper` |
| Price | $0.002 per keyword delivered ($2 per 1,000; a comparison of N keywords counts N); $0.0009 per Trending now topic; failures and countries with no result are free |
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

## Compare keywords on one scale

Put up to 5 keywords in **one** `searchTerms` entry, separated by commas. The result is one object with `comparedTerms`, `interestOverTime` (each point has `values` per term), `averages`, `terms` (average, peak date and trend change of each) and `leader`:

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~google-trends-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"searchTerms":["bitcoin, ethereum, solana"],"geo":"US","timeRange":"today 12-m"}'
```

Set `splitByComma: true` if a comma is part of one keyword and should not start a comparison.

## Trending now (no keyword)

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~google-trends-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"mode":"trending","trendingGeo":["US","BR"],"trendingHours":"24","trendingMaxItems":20,"includeNews":true}'
```

One object per trending search: `title`, `searchVolume`, `searchVolumeText` ("500K+"), `increasePercent`, `startedAt`, `endedAt`, `isActive`, `categories`, `relatedQueries`, `articles`, `picture`, `rank`, `exploreUrl`. `trendingHours` is `4`, `24`, `48` or `168`; `trendingCategories` takes names or ids (`Sports`, `Technology`, `17`); `activeOnly: true` keeps only searches still trending.

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
| `includeInterestByMetro` | US searches: interest by Nielsen metro area (210 DMAs), in `interestByMetro` |
| `includeInterestOverTime` | Off to skip the time series and save time |
| `spreadsheetId` | A public Google Sheet; the first column is read as keywords |
| `viewedFrom` | Country the request comes from (for local results) |
| `maxResults` | Cap each list (regions, metros, related searches) |
| `outputFormat` | `summary` (default, one object per keyword) or `flat` (one row per data point, for spreadsheets) |
| `language`, `timezoneOffset` | Language of names and dates (`en-US`, `pt-BR`); minutes from UTC |

## Output fields worth keeping

`searchTerm`, `geo`, `timeRange`, `interestOverTime` (list of `date` and `value`, 0 to 100), `average`, `peakDate`, `trendChangePercent`, `interestByRegion`, `interestByMetro`, `relatedQueriesTop`, `relatedQueriesRising` (with "Breakout" values), `trendsUrl`, `hasData`; in comparisons also `comparedTerms`, `terms` and `leader`.

Related topics: Google withholds them from automated requests; the result says `relatedTopicsStatus: withheld_by_google`. Do not promise them to the user.

Google Trends values are relative (100 = the peak in that period and place), not search counts. Keywords from separate results are not on the same scale; to compare them, use one comparison entry (above).

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| `hasData: false` | Too little search volume for that keyword, place and period | Widen the period or the place |

## Cost check before big batches

Cost is `keywords x $0.002`, plus `trending topics x $0.0009`. Above 2,000 keywords ($4), tell the user the estimate and wait for a yes. Cap a run with `maxTotalChargeUsd`.

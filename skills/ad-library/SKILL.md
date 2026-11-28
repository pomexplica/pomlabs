---
name: ad-library
description: Gets the ads any advertiser runs on Google (Search, Display, YouTube, Shopping) from the Google Ads Transparency Center: ad id, format, first and last shown, days shown, preview and image links. Search by domain, advertiser name or advertiser id, in any country. Use when the user asks what ads a competitor runs on Google, for how long, or wants creative inspiration or to see who advertises in a market. Runs on Apify with the user's own APIFY_TOKEN; pay per ad delivered.
license: MIT
---

# Ad library (Google Ads Transparency)

One Apify Actor by Pom Labs returns the ads an advertiser runs on Google, from the public Ads Transparency Center. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~google-ads-transparency-scraper` |
| Price | $0.0015 per ad delivered ($1.5 per 1,000); names that match nothing, advertisers with no ads and blocked requests are free |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | Seconds to a minute for 20 to 100 ads |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/google-ads-transparency-scraper if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/google-ads-transparency-scraper`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get ads

```bash
# advertisers: a domain, an advertiser name or an advertiser id (AR...)
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~google-ads-transparency-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"advertisers":["mercadolivre.com.br"],"region":"BR","maxAdsPerAdvertiser":20}'
```

The response is a JSON array with one object per ad.

## Input fields

| Field | Meaning |
| --- | --- |
| `advertisers` (required) | Domains (`nike.com`), advertiser names, advertiser ids (`AR...`) or Transparency Center links |
| `region` | 2-letter country code (`BR`, `US`, `DE`...) or `anywhere`. Default `BR` |
| `maxAdsPerAdvertiser` | Default 40; `0` = all. This is the cost cap: always set it, big advertisers have hundreds of thousands of ads |
| `format` | Only `text`, `image` or `video` ads |
| `platform` | Only ads shown on one Google platform |
| `dateFrom`, `dateTo` | Only ads shown in this date range (`YYYY-MM-DD`) |

There are more fields on the Actor's page (https://apify.com/pomlabs/google-ads-transparency-scraper); these are the main ones.

## Output fields worth keeping

`advertiserName`, `advertiserId`, `adId`, `format`, `firstShown`, `lastShown`, `daysShown`, `adUrl`, `previewUrl`, `imageUrl`, `domain`, `region`, `totalAdsRange`.

An ad shown for many days is a hint that it works, not proof. The Transparency Center shows no spend, clicks or targeting, so do not claim them.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Empty result for a name | No advertiser matches that name | Try the domain or the advertiser id; these rows are free |
| Empty result for a region | The advertiser has no ads there | Try `anywhere` or another region |

## Cost check before big batches

Cost is `advertisers x maxAdsPerAdvertiser x $0.0015` at most. Above $5 (about 3,300 ads), tell the user the estimate and wait for a yes.

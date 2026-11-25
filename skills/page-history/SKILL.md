---
name: page-history
description: Gets past versions of any web page from the Wayback Machine, Arquivo.pt, archive.today and Common Crawl as clean text, with the lines added and removed between versions, an optional AI summary of what changed, and a check of whether the page is still online. Use when the user asks what a page (pricing, terms, a competitor's site, a deleted post or profile) said before, when something changed on it, or wants to recover text from a page that is gone. Runs on Apify with the user's own APIFY_TOKEN; pay per version.
license: MIT
---

# Page history (web archives)

One Apify Actor by Pom Labs looks a page up in four web archives and returns its versions as text. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~page-time-machine` |
| Price | $0.0025 per version delivered ($2.50 per 1,000); $0.005 per AI summary of a change; the live check is free |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | 5 to 100 s per page (median about 60 s in our runs); archives are slow |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/page-time-machine if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/page-time-machine`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get versions

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~page-time-machine/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"urls":["https://apify.com/pricing"],"from":"2024","to":"2026","maxVersionsPerUrl":3,"includeChanges":true,"aiSummary":false}'
```

The response is a JSON array: items with `type` `version` (one per version) and one item with `type` `page` per URL (what was found in each archive and the live check).

## Input fields

| Field | Meaning |
| --- | --- |
| `urls` (required) | Pages to look up (full URL or domain/path) |
| `from`, `to` | `YYYY`, `YYYY-MM` or `YYYY-MM-DD`; empty = whole history |
| `maxVersionsPerUrl` | Default 3, spread over the period (always the first and the last); identical texts are skipped and not charged |
| `sources` | Any of `wayback`, `arquivopt`, `archivetoday` (default), and `commoncrawl` |
| `includeText`, `includeChanges` | On by default |
| `aiSummary` | 1 to 4 bullets on what really changed, extra charge per summary |
| `keyword` | Keep only versions whose text contains it |
| `checkLive` | On by default: online, deleted (404/410) or redirected today |
| `maxTextChars` | Default 20,000; longer texts are cut (`textTruncated`) |

## Output fields worth keeping

Version items: `url`, `source`, `capturedAt`, `archiveUrl`, `title`, `text`, `textLength`, `changedFrom`, `changes` (`added`, `removed`, `linesAdded`, `linesRemoved`, `similarity`), `aiSummary`. Page items: `capturesFound`, `versionsReturned`, `duplicatesSkipped`, `archiveToday`, `waybackCaptures`.

Always give the `archiveUrl` with a quote, so the user can open the saved copy. A page that was never archived has no history here.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| `versionsReturned: 0` | No archive saved that page in the period, or `keyword` filtered everything | Widen `from`/`to`, drop `keyword`, or add `commoncrawl` |

## Cost check before big batches

Cost is `urls x maxVersionsPerUrl x $0.0025`, plus $0.005 per AI summary. Above $5, tell the user the estimate and wait for a yes.

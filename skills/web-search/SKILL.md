---
name: web-search
description: Searches Google, Bing, Yahoo, DuckDuckGo and Brave in one call and returns one merged list with real URLs and how many engines agree, plus Google's AI Overview, People Also Ask, related searches, ads, optional page text as Markdown, and optional answers from AI assistants with their sources. Use when the user needs web search results with country, date or site filters, wants to compare engines, check which pages AI assistants cite, or read the text of the top results. Runs on Apify with the user's own APIFY_TOKEN; pay per engine that answered.
license: MIT
---

# Web search (several engines)

One Apify Actor by Pom Labs asks several search engines at once and returns one merged result list. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~multi-engine-web-search` |
| Price | $0.0025 per engine that answered ($2.5 per 1,000); $0.002 per page read as Markdown ($2 per 1,000) and per AI assistant answer or lead found when those options are on (see the Actor's page); blocked or empty engines are free |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | Seconds per query with Google and Bing; more with extra engines or page reading |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/multi-engine-web-search if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/multi-engine-web-search`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Search

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~multi-engine-web-search/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"queries":["best running shoes 2026"],"engines":["google","bing"],"maxResultsPerEngine":10}'
```

The response is a JSON array with one object per query.

## Input fields

| Field | Meaning |
| --- | --- |
| `queries` | One search per line, up to 20. A full address (`https://...`) is read directly |
| `engines` | `google`, `bing`, `yahoo`, `duckduckgo`, `brave` (default: Google and Bing) |
| `country`, `language` | Two-letter codes for where and in what language to search |
| `timeRange`, `dateFrom`, `dateTo` | Only pages from a period or a date range |
| `maxResultsPerEngine` | 1 to 10 (default 10) |
| `maxPagesPerQuery` | 1 to 5 result pages per engine; each extra page is another engine charge |
| `includeDomains`, `excludeDomains` | Keep or drop sites |
| `forceExactMatch` | Search the query as an exact phrase |
| `fileTypes` | Only files such as `pdf`, `xlsx` |
| `fetchContent` | Read the main text of the top 1 to 10 pages as Markdown. Extra charge |
| `mobileResults` | Search as a phone |

There are more fields on the Actor's page (https://apify.com/pomlabs/multi-engine-web-search); these are the main ones.

## Output fields worth keeping

Per query: `status`, `enginesAnswered`, `results` (title, real URL, `snippet`, `engines` that returned it), `aiOverview`, `peopleAlsoAsk`, `relatedQueries`, `paidResults`, and `content` when pages are read. AI assistant answers come in `aiAnswers` with the sources they cited, when requested.

Give the URL with each claim. Search results are what the engines showed at that moment, from the chosen country and language.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| An engine shows `blocked` or `empty` | The engine did not answer | Not charged; retry later or use another engine |

## Cost check before big batches

Cost is `queries x engines x pages x $0.0025` at most, more with page reading or AI answers. Above $5, tell the user the estimate and wait for a yes.

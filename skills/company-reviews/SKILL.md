---
name: company-reviews
description: Gets Trustpilot reviews of any company as JSON: rating, title, text, language, dates, likes, reviewer and the company's reply, with a free company row (TrustScore, total reviews, Trustpilot's AI summary). Also searches companies, browses categories and can run an AI pain-point analysis. Use when the user asks what customers say about a company, wants complaints or praise by star, language or date, wants to compare competitors, or track new reviews. Runs on Apify with the user's own APIFY_TOKEN; pay per review delivered.
license: MIT
---

# Company reviews (Trustpilot)

One Apify Actor by Pom Labs returns the Trustpilot reviews of companies, and can also search companies and analyze complaints. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~trustpilot-reviews-scraper` |
| Price | $0.0005 per review delivered ($0.5 per 1,000); $0.00022 per company row ($0.23 per 1,000) for company search, categories and company info; review insights, AI pain-point analysis and contact emails have their own events listed on the Actor's page; the company row of the reviews mode is free |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | Seconds for 40 reviews; longer for thousands |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/trustpilot-reviews-scraper if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/trustpilot-reviews-scraper`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

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
# companies: a website (amazon.com), a Trustpilot URL, or a reviewer profile URL; "mode" defaults to reviews
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~trustpilot-reviews-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"companies":["www.amazon.com"],"maxReviews":40,"stars":["1","2"]}'
```

The response is a JSON array: one object per review, plus a free company row per company, marked by `type`.

## Input fields

| Field | Meaning |
| --- | --- |
| `mode` | `reviews` (default), `searchCompanies`, `categories` or `companyInfo` |
| `companies` | Websites, Trustpilot URLs or reviewer profile URLs, one per line |
| `searchTerms` | Brand names to resolve to a Trustpilot page, or keywords in `searchCompanies` mode |
| `maxReviews` | Per company, default 40; `0` = as many as exist. This is the cost cap: always set it |
| `sort` | `recency` (default) or most relevant |
| `stars` | Only these star ratings |
| `languages` | Two-letter review languages (`en`, `pt`, `de`) |
| `lastDays`, `startDate` | Only reviews from a recent window or a date |
| `topics` | Trustpilot topics such as `customer_service`, `delivery_service`, `refund` |
| `sampling` | `balanced` takes the same number from each star when the limit is below the review count |

There are more fields on the Actor's page (https://apify.com/pomlabs/trustpilot-reviews-scraper); these are the main ones.

## Output fields worth keeping

Review rows: `rating`, `title`, `text`, `language`, `publishedAt`, `likes`, `verified`, reviewer name and country, the company reply, `suspicionScore`, `url`. Company row: TrustScore, total reviews, star breakdown, categories, reply rate.

Trustpilot reviews are self-selected. When summarizing, say how many were read and quote them exactly.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Company not found | The website or brand does not match a Trustpilot page | Try the Trustpilot URL; unknown companies are free |

## Cost check before big batches

Cost is `companies x maxReviews x $0.0005` at most. Above 10,000 reviews ($5), tell the user the estimate and wait for a yes.

---
name: company-pages
description: Gets the public LinkedIn page of companies as JSON - description, industry, size, employees on LinkedIn, followers, headquarters, website, founding year, specialties, recent posts and similar companies. Use when the user names a company or a linkedin.com/company link and wants firmographics, to enrich a list of accounts or leads, to size or qualify a company, to see what it has been posting, or to find similar companies. Runs on Apify with the user's own APIFY_TOKEN; pay per company, no LinkedIn login or cookies.
license: MIT
---

# Company pages (LinkedIn)

One Apify Actor by Pom Labs returns what a logged-out visitor sees on a LinkedIn company page. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~linkedin-company-scraper` |
| Price | $0.003 per company ($3 per 1,000) |
| Auth | `Authorization: Bearer $APIFY_TOKEN`. No LinkedIn account or cookie is used |
| Typical time | 3 to 10 s per company |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/linkedin-company-scraper if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/linkedin-company-scraper`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get companies

The input is the company's LinkedIn slug (the part after `/company/`) or the full URL. A bare company name only works when it equals the slug; if the user gives a name, find the LinkedIn URL first (web search) instead of guessing. Keep each call to 25 companies or fewer.

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~linkedin-company-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"companies":["https://www.linkedin.com/company/apify/"],"includeRecentPosts":true}'
```

The response is a JSON array with one object per company.

## Input fields

| Field | Meaning |
| --- | --- |
| `companies` (required) | Array of company page URLs or slugs. Showcase and school pages work too |
| `includeRecentPosts` | `true` (default) adds the latest public posts. Set `false` for plain enrichment to save tokens |

## Output fields worth keeping

`status` (`ok` or an error code), `name`, `description`, `industry`, `companySize` (LinkedIn's range, e.g. `201-500 employees`), `employeesOnLinkedIn` (members who list the company), `followers`, `founded`, `headquarters`, `address`, `locations`, `website`, `specialties`, `companyType`, `linkedinUrl`, `similarCompanies` (name and URL) and `recentPosts` (`date`, `text`, likes, link).

`companySize` is what the company declared; `employeesOnLinkedIn` is a count of profiles. Report both when headcount matters, and name which is which.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Item with `status: "not_found"` | No company page under that slug | Find the exact LinkedIn URL and run again |
| Item with another `status` other than `ok` | LinkedIn kept showing its login wall | Retry that company once, then report the item's `error` text |
| A different company than expected | The slug was guessed from the name | Find the exact LinkedIn URL and run again |

## Cost check before big batches

Cost is `companies x $0.003`. Above 1,000 companies, tell the user the estimate and wait for a yes.

---
name: business-leads
description: Gets businesses from Google Maps by search, city, area or link: phone (mobile and WhatsApp flagged), website and its type, address, hours, rating, owner claim, with a free filter for businesses without a website, optional emails and social profiles from their sites and Brazilian company (CNPJ) data. Use when the user wants a list of local businesses, leads for outreach or a web agency, a competitor map, or the details of a place. Runs on Apify with the user's own APIFY_TOKEN; pay per place delivered.
license: MIT
---

# Business leads (Google Maps)

One Apify Actor by Pom Labs searches Google Maps and returns businesses with their contact details, as JSON. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~google-maps-business-leads` |
| Price | $0.0036 per place delivered ($3.6 per 1,000); $0.0018 per contacts found (emails or social profiles from the website, only when found) ($1.8 per 1,000) and $0.0045 per Brazilian company record ($4.5 per 1,000) when those options are on; other options have their own events listed on the Actor's page; filters and errors are free |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | About 1 s for 20 places; about 30 s for 100 places with website contacts |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/google-maps-business-leads if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/google-maps-business-leads`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Search places

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~google-maps-business-leads/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"queries":["plumber austin tx"],"maxResults":20,"language":"en","withoutWebsite":false}'
```

The response is a JSON array with one object per place.

## Input fields

| Field | Meaning |
| --- | --- |
| `queries` | What you would type in Google Maps, like `dentist austin tx`. Up to 50 |
| `startUrls` | Google Maps place links, place ids or cids, to read places without a search |
| `maxResults` | 1 to 200 per search. This is the cost cap: always set it |
| `location` | Added to every search, like `Austin, TX` |
| `city`, `state`, `countryCode` | Search inside the real limits of a city, state or country |
| `latitude`, `longitude`, `radiusMeters` | Search around a point |
| `scanArea` | Split the area into cells to pass the 200-place limit (up to 1,000) |
| `language` | Language of categories and hours: `en`, `pt-BR`, `es`... |
| `withoutWebsite` | Free filter: only places with no website of their own |
| `onlyMobilePhones`, `onlyWithPhone` | Free filters on the phone |
| `minRating` | Free filter, like `4.5` |
| Contacts and company data | Emails and social profiles from the website, and Brazilian CNPJ data, are separate switches on the Actor's page, charged only when something is found |

There are more fields on the Actor's page (https://apify.com/pomlabs/google-maps-business-leads); these are the main ones.

## Output fields worth keeping

`name`, `category`, `rating`, `reviewsCount`, `phone`, `isMobilePhone`, `whatsapp`, `website`, `websiteType`, `hasOwnWebsite`, address parts, `latitude`, `longitude`, `openingHours`, `claimedByOwner`, `placeId`, `googleMapsUrl`, and `emails` when contacts are on.

These are business listings that are public on Google Maps. Use them for what the user described, and tell the user that outreach rules (such as anti-spam laws) are theirs to follow.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Fewer places than `maxResults` | Google Maps lists about 200 places per search | Use `scanArea`, `city` or several `locations` |
| Few emails | Many sites show none | Contacts are charged only when found |

## Cost check before big batches

Cost is `queries x maxResults x $0.0036` at most, plus $0.0018 per place with contacts found when that option is on. Above $5, tell the user the estimate and wait for a yes.

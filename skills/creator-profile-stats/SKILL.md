---
name: creator-profile-stats
description: Gets the public numbers of Instagram profiles as JSON - followers, bio, links, verification - and, on request, the views of the latest Reels with the account's median, so each Reel can be judged against the account's own baseline. Use when the user names an Instagram account or profile link and wants its follower count, bio or links, wants to know how its Reels usually perform, which recent Reel beat the account's normal, or wants to compare creators or vet an influencer. Runs on Apify with the user's own APIFY_TOKEN; pay per profile, no login to Instagram.
license: MIT
---

# Creator profile stats (Instagram)

One Apify Actor by Pom Labs returns the public data of Instagram profiles. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~instagram-profile-data` |
| Price | $0.0025 per profile; + $0.003 per profile when Reels stats are requested |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | 5 to 15 s per profile; 30 to 60 s with Reels stats |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/instagram-profile-data if they matter to the user.

## No terminal? Use the Apify connector

If the Apify connector (MCP server `mcp.apify.com`) is connected, skip the token setup and the `curl` call: call `call-actor` with `actor` set to the Actor name from the table (with `/` instead of `~`) and `input` set to the same JSON as below, then read the result with `get-dataset-items` and the `datasetId` it returns. The connector bills the user's own Apify account at the same prices.

## Setup (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get profiles

Keep each call to 5 profiles or fewer when `includeReelsStats` is on, 25 otherwise, so it finishes inside the 300 s sync limit.

```bash
# Usernames (with or without @) or profile URLs
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~instagram-profile-data/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"usernames":["nasa"],"includeReelsStats":true}'
```

The response is a JSON array with one object per profile.

## Input fields

| Field | Meaning |
| --- | --- |
| `usernames` (required) | Array of usernames or profile URLs. Duplicates are removed |
| `includeReelsStats` | `true` adds `reelsStats`. Leave `false` when the user only wants followers, bio or links |

## Output fields worth keeping

`status` (`ok` or an error code), `username`, `fullName`, `followers`, `following`, `biography`, `externalUrl`, `bioLinks`, `isVerified`, `isPrivate`, `profileUrl`.

`reelsStats`, when requested: `reelsRead` (about 15), `medianViews`, `averageViews`, `maxViews`, and `reels`, each with `url`, `views`, `viewsText` (as Instagram shows it, e.g. `2.6M`) and `multipleOfMedian`.

## Reading the numbers

- Judge a Reel by `multipleOfMedian`, not by raw views: 2.6M views is weak on an account whose median is 4.1M (0.63x) and huge on one whose median is 50K.
- Views come rounded as Instagram displays them, so treat them as two significant digits.
- `provisional: true` in `reelsStats` means few Reels were readable; say so instead of presenting the median as solid.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Item with `status: "not_found_or_restricted"` | The profile does not exist, changed its name, or Instagram shows it only to logged-in users (age or region restricted) | Report it for that username, with the item's `error` text, and continue with the others |
| `isPrivate: true` and no `reelsStats` | Private account | Report followers and bio only; Reels of private accounts are not readable |

## Cost check before big batches

Cost is `profiles x $0.0025`, or `profiles x $0.0055` with Reels stats. Above 500 profiles, tell the user the estimate and wait for a yes.

---
name: creator-profile-stats
description: Gets the public numbers of Instagram profiles as JSON - exact followers and posts count, bio, links, category, business address, the 12 latest posts with likes and comments, engagement rate - and, on request, related accounts, the About this account section (date joined, country) and the exact views of the latest Reels with the account's median. Use when the user names an Instagram account or profile link and wants its follower count, posts, engagement rate, bio or links, similar accounts, when it was created or where it is based, how its Reels usually perform or which recent Reel beat the account's normal, or wants to compare creators or vet an influencer. Runs on Apify with the user's own APIFY_TOKEN; pay per profile, no login to Instagram.
license: MIT
---

# Creator profile stats (Instagram)

One Apify Actor by Pom Labs returns the public data of Instagram profiles. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~instagram-profile-data` |
| Price | $0.0025 per profile (latest posts, engagement and related accounts included); + $0.0063 with About this account; + $0.003 with Reels stats |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | 5 to 15 s per profile, several in parallel |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/instagram-profile-data if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/instagram-profile-data`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get profiles

Keep each call to 25 profiles or fewer, so it finishes inside the 300 s sync limit.

```bash
# Usernames (with or without @) or profile URLs
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~instagram-profile-data/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"usernames":["nasa","starbucks"],"includeRelatedProfiles":true,"includeAboutSection":true,"includeReelsStats":true}'
```

The response is a JSON array with one object per profile.

## Input fields

| Field | Meaning |
| --- | --- |
| `usernames` (required) | Array of usernames, profile URLs or numeric profile ids. Duplicates are removed |
| `includeLatestPosts` | Default `true`: `latestPosts` (12) and `postStats` (engagement). Included in the profile price |
| `includeRelatedProfiles` | `true` adds `relatedProfiles` (accounts Instagram shows as similar); `maxRelatedProfiles` 1 to 80, default 30. Included in the profile price |
| `includeAboutSection` | `true` adds `about` (date joined, country, former usernames). Extra charge |
| `includeReelsStats` | `true` adds `reelsStats`. Extra charge |

## Output fields worth keeping

`status` (`ok` or an error code), `username`, `fullName`, `followers`, `following`, `postsCount`, `biography`, `externalUrl`, `bioLinks`, `isVerified`, `isPrivate`, `category`, `accountType` (`personal`, `business`, `creator`), `businessAddress`, `highlights`, `threadsUsername`, `profileUrl`.

`latestPosts`: `url`, `type`, `caption`, `timestamp`, `likesCount`, `commentsCount`, `isPinned`, `location`, `taggedUsers`, `coauthors`.

`postStats`: `avgLikes`, `medianLikes`, `avgComments`, `engagementRate` (percent of followers per post, pinned posts left out), `postsPerWeek`, `daysSinceLastPost`.

`relatedProfiles` and `about` (`dateJoined`, `accountBasedIn`), when requested.

`reelsStats`, when requested: `reelsRead` (12), `medianViews`, `averageViews`, `maxViews`, and `reels`, each with `url`, exact `views`, `likes`, `comments`, `pinned` and `multipleOfMedian` (pinned Reels are left out of the median).

## Reading the numbers

- Judge a Reel by `multipleOfMedian`, not by raw views: 2.6M views is weak on an account whose median is 4.1M (0.63x) and huge on one whose median is 50K.
- Engagement rate under 1% is normal for accounts with millions of followers; compare it with accounts of the same size.
- `provisional: true` in `reelsStats` means few Reels were readable; say so instead of presenting the median as solid.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Item with `status: "not_found_or_restricted"` | The profile does not exist, changed its name, or Instagram shows it only to logged-in users (age or region restricted) | Report it for that username, with the item's `error` text, and continue with the others; it was not charged |
| `isPrivate: true` and no `reelsStats` | Private account | Report followers and bio only; Reels of private accounts are not readable |

## Cost check before big batches

Cost is `profiles x $0.0025`, plus $0.0063 per profile with About this account and $0.003 with Reels stats. Above 500 profiles, tell the user the estimate and wait for a yes.

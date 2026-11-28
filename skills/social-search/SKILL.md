---
name: social-search
description: Searches Facebook without login: recent posts by keyword, posts of public groups with group details, comments and replies, buy-and-sell listings with price, people by name in a city, pages, places and group members. Use when the user wants to find what people post about a topic on Facebook, find leads or listings, research a group, or look up a person or page by name. Runs on Apify with the user's own APIFY_TOKEN; pay per result.
license: MIT
---

# Social search (Facebook)

One Apify Actor by Pom Labs searches Facebook for posts, groups, people, pages and listings and returns the results as JSON. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~facebook-search-scraper` |
| Price | the main events are $0.001 per post ($1 per 1,000), $0.001 per profile ($1 per 1,000), $0.001 per page ($1 per 1,000), $0.0005 per group post ($0.5 per 1,000), $0.002 per group or member ($2 per 1,000) and $0.0002 per comment or reply ($0.2 per 1,000); contact finding, AI labels and listing locations have their own small events listed on the Actor's page; errors and empty queries are free |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | Seconds to a minute for 20 to 100 results |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/facebook-search-scraper if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/facebook-search-scraper`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

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
# mode: posts, top, people, pages, places, events, videos, group_posts, post_urls, following, city_pages, groups or members
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~facebook-search-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"mode":"posts","queries":["creatina"],"maxResults":10}'
```

The response is a JSON array: one object per result, marked by `type`.

## Input fields

| Field | Meaning |
| --- | --- |
| `mode` | `posts`, `top`, `people`, `pages`, `places`, `events`, `videos`, `group_posts`, `post_urls`, `following`, `city_pages`, `groups` or `members` |
| `queries` | Keywords, a city name, group links or ids, or post links, depending on the mode. Up to 20 |
| `maxResults` | Results per query, default 20. This is the cost cap: always set it. Facebook ends a keyword search near 100 posts |
| `recentPosts` | Newest first (default on) |
| `startDate`, `endDate` | Only posts in this range (`2026-11-01` or `7 days`) |
| `city` | For `people`: a Brazilian city by name or a Facebook city id |
| `includeComments`, `maxCommentsPerPost` | Comments of each post, charged per comment |
| `includeGroupDetails` | The group's About page: member count, rules, admins |
| `extractContacts`, `onlyWithContact` | Emails and phones written in posts |
| `onlyListings`, `listingStatus` | Buy-and-sell posts with price and seller |
| `keywords`, `excludeKeywords` | Keep or drop posts by words, before you pay for them |
| `onlyNew`, `stateKey` | Monitor: only posts never delivered before |

There are more fields on the Actor's page (https://apify.com/pomlabs/facebook-search-scraper); these are the main ones.

## Output fields worth keeping

Posts: `url`, `text`, `publishedAt`, `authorName`, `reactionsCount`, `commentsCount`, `postType`, `engagementScore`. Group rows: `memberCount`, `rules`, `admins`. Listings: `title`, `price`, `currency`, `isSold`, `location`. People and pages: name, link and the details Facebook shows.

These are public results that match a search, not a complete census. Say how many were read, and treat personal data (names, contacts) with care: use it only for what the user described.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Fewer results than `maxResults` | Facebook ends keyword searches near 100 posts | Narrow by date or split the query |
| Empty array for a group | The group is private or the link is wrong | Say so; empty queries are free |

## Cost check before big batches

Cost is `queries x maxResults x $0.001` at most for posts, and more with comments, contacts or labels on. Above $5, tell the user the estimate and wait for a yes.

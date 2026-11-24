---
name: short-post-search
description: Gets public Threads posts and profiles as JSON - posts found by a keyword or hashtag search, the latest posts of a profile, or a single post with its replies - with likes, replies, reposts, quotes and author. Use when the user wants to know what people are posting on Threads about a topic, brand or product, wants a Threads profile's follower count and recent posts, wants the replies of a Threads post, or shares a threads.com or threads.net link. Runs on Apify with the user's own APIFY_TOKEN; pay per post or profile, no Threads or Instagram login.
license: MIT
---

# Short post search (Threads)

One Apify Actor by Pom Labs returns the public posts and profiles Threads shows to a logged-out visitor. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~threads-scraper` |
| Price | $0.002 per post; $0.002 per profile. Duplicates are never charged twice |
| Auth | `Authorization: Bearer $APIFY_TOKEN`. No Threads or Instagram account is used |
| Typical time | 15 to 40 s per call |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/threads-scraper if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to `pomlabs/threads-scraper` and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get posts and profiles

Give at least one of `searchQueries`, `profiles` or `postUrls`. `maxItems` is the cost cap for posts: always set it.

```bash
# What people post about a topic
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~threads-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"searchQueries":["running shoes"],"maxItems":40}'
```

```bash
# A profile and its latest posts
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~threads-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"profiles":["zuck"],"includeProfilePosts":true,"maxItems":10}'
```

The response is a JSON array; each item has `type` `profile` or `post`.

## Input fields

| Field | Meaning |
| --- | --- |
| `searchQueries` | Keywords or hashtags, searched as in the app. Each search returns the public posts Threads shows for it, usually 20 to 60 |
| `profiles` | Usernames or profile URLs (`threads.com/@username` or `threads.net/@username`) |
| `includeProfilePosts` | `true` (default) also returns the latest posts on each public profile page, usually 4 to 10 |
| `postUrls` | Links to single posts. Returns the post and the replies shown on its public page |
| `maxItems` | Stop after this many posts in total (0 = no limit). Profiles are not counted |

## Output fields worth keeping

Posts: `text`, `url`, `createdAt`, `likes`, `replies`, `reposts`, `quotes`, `shares`, and `author` (`username`, `fullName`, `isVerified`). Profiles: `username`, `fullName`, `biography`, `followers`, `isVerified`, `isPrivate`, `bioLinks`, `url`, `status`.

A search returns what Threads chooses to show, not every post on the topic, and a profile page shows only its latest few posts: say how many posts were read and do not present them as the full picture.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Profile item with `status` other than `ok` | The username does not exist or its page could not be read | Report it for that username with the item's `error` text; usernames that do not exist are not charged |
| Few or no posts for a search | Threads shows little for that wording | Try the wording people would use in a post, or a hashtag; say what was searched |

## Cost check before big batches

Cost is at most `maxItems x $0.002` plus `profiles x $0.002`. Above 2,500 posts ($5), tell the user the estimate and wait for a yes.

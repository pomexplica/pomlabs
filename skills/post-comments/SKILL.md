---
name: post-comments
description: Gets the comments of public Instagram Reels and posts as JSON: text, author, verified badge, likes, reply count, date and link, with optional replies. No Instagram login. Use when the user shares an Instagram post or Reel link and wants what people say, the most liked comments, questions or objections, sentiment, or every comment of a giveaway. Runs on Apify with the user's own APIFY_TOKEN; pay per comment delivered.
license: MIT
---

# Post comments (Instagram)

One Apify Actor by Pom Labs returns the comments of public Instagram posts and Reels, one row per comment. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~instagram-comments-scraper` |
| Price | $0.001 per comment delivered ($1 per 1,000); private or deleted posts, posts without comments and errors are free |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | Seconds for 100 comments; up to 1,000 comments per post |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/instagram-comments-scraper if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/instagram-comments-scraper`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get comments

```bash
# Post or Reel links, post ids, or a profile link / @username (reads comments of its latest posts)
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~instagram-comments-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"urls":["https://www.instagram.com/reel/DFFyRp-pINJ/"],"maxComments":100}'
```

The response is a JSON array with one object per comment.

## Input fields

| Field | Meaning |
| --- | --- |
| `urls` | Public post or Reel links, post ids, or profile links or @usernames |
| `maxComments` | Per post, 1 to 1,000, default 100. This is the cost cap: always set it |
| `maxItems` | Optional cap on comments for the whole run |
| `includeReplies` | Also return the replies of each comment, inside the comment |
| `maxRepliesPerComment` | Default 20, when replies are on |
| `sortOrder` | Most liked or newest first (the Actor reads up to 1,000 comments to sort them) |
| `maxPostsPerProfile` | For profile links: how many latest posts to read (default 3) |
| `continueOnDuplicates` | By default a post given twice is read once |

There are more fields on the Actor's page (https://apify.com/pomlabs/instagram-comments-scraper); these are the main ones.

## Output fields worth keeping

`text`, `author`, `authorFullName`, `authorVerified`, `likeCount`, `replyCount`, `createdAt`, `commentUrl`, `postUrl`, `shortcode`, `isRanked`, and `replies` when requested.

When summarizing, quote comments exactly and say how many were read: the first 100 comments are not the whole audience.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Empty result for a post | The post is private or deleted, or has no comments | Say so for that post; these are not charged |

## Cost check before big batches

Cost is `posts x maxComments x $0.001` at most. Above 5,000 comments ($5), tell the user the estimate and wait for a yes.

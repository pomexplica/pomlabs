---
name: community-research
description: Gets Reddit posts and comments as JSON from subreddits, post links, user profiles or a search across Reddit, with date filters going back years and a flag on what moderators removed. Use when the user wants to know what people say about a product, problem or topic on Reddit, wants real user quotes, complaints, feature requests or reviews, wants the threads of a subreddit over a period, the comments of a thread, or a user's history. Runs on Apify with the user's own APIFY_TOKEN; pay per item, no Reddit login or API key.
license: MIT
---

# Community research (Reddit)

One Apify Actor by Pom Labs returns Reddit posts and comments. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~reddit-posts-comments-scraper` |
| Price | $0.002 per post or comment delivered ($2 per 1,000); $0.02 per subreddit or user report |
| Auth | `Authorization: Bearer $APIFY_TOKEN`. No Reddit account or API key is used |
| Typical time | 10 to 30 s for up to 100 items |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/reddit-posts-comments-scraper if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/reddit-posts-comments-scraper`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get posts and comments

`maxItems` is the cost cap: always set it. Keep each call to 500 items or fewer so it finishes inside the 300 s sync limit.

```bash
# What a subreddit said about a topic since a date, with the comments
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~reddit-posts-comments-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"startUrls":["https://www.reddit.com/r/webscraping"],"keyword":"proxy","dateFrom":"2026-01-01","includeComments":true,"maxCommentsPerPost":20,"maxItems":100}'
```

```bash
# Search all of Reddit (up to 25 posts per search, by relevance)
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~reddit-posts-comments-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"searchQuery":"best youtube transcript api","includeComments":true,"maxCommentsPerPost":10,"maxItems":100}'
```

The response is a JSON array of items; posts and comments are separate items.

## Input fields

| Field | Meaning |
| --- | --- |
| `startUrls` | Subreddit URLs (or `r/name`), post URLs and user profile URLs, mixed freely. Leave empty to only search |
| `searchQuery` | Searches all of Reddit by relevance, up to 25 posts per search |
| `keyword` | Only posts of the given subreddits whose title or text contains these words (last 12 months unless `dateFrom` is set) |
| `dateFrom`, `dateTo` | `YYYY-MM-DD` |
| `sort` | Order of posts read from subreddits and users; default `newest` |
| `maxItems` | Total posts plus comments, default 100 |
| `maxPostsPerSource` | Posts per subreddit, user or search, default 100 |
| `includeComments`, `maxCommentsPerPost`, `maxCommentDepth` | Comments as separate items; depth 0 is top-level only |
| `removedFilter` | Default `all`. Every item says whether it was removed and by whom |
| `includeSubredditReport`, `includeUserReport` | One extra item with rules and top posters of a subreddit, or the totals and subreddits of a user ($0.02 each) |

## Output fields worth keeping

`submissionType` (`post`, comment or a report type), `title`, `bodyText`, `author`, `subreddit`, `score`, `upvoteRatio`, `numComments`, `submittedAt`, `submissionUrl`, `postFlair`, `isRemoved`, `removedBy`, `isEdited`.

- Posts come from an archive, so `score` is the score at capture time (`scoreCapturedAt`), not always today's.
- Check `isRemoved` before quoting: a removed post may still carry its original text (`originalTextKept`), and the user should know it was removed.
- Quote exactly, link each quote with `submissionUrl`, and say how many items were read.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| A single item with `status: "error"` (message such as `Timeout. Maybe slow down a bit`) | The archive behind the Actor was busy | Wait a few seconds and retry the same call once |
| `run-failed` (HTTP 400) on a subreddit or user | The name does not exist or is misspelled | Check the name with the user; do not retry |
| Empty array | No post matched the keyword, dates or search | Widen the dates or drop `keyword`; say what was searched |
| Only 25 posts from a search | That is the search limit | Use `startUrls` with the relevant subreddits plus `keyword` for more |

## Cost check before big batches

Cost is at most `maxItems x $0.002`, plus $0.02 per report. Above 2,500 items ($5), tell the user the estimate and wait for a yes.

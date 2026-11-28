---
name: page-posts
description: Gets the public posts of any Facebook page as JSON: text, date, link, total reactions and each reaction type, comments, shares, photo and video links, with optional page details, comments and video captions. No Facebook login. Use when the user gives a Facebook page (link, name or id) and wants its recent posts, what performed best, or to monitor what a brand or competitor posts. Runs on Apify with the user's own APIFY_TOKEN; pay per post delivered.
license: MIT
---

# Page posts (Facebook)

One Apify Actor by Pom Labs returns the public posts of Facebook pages, newest first, with their reactions, comments and shares. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~facebook-page-posts-scraper` |
| Price | $0.002 per post delivered ($2 per 1,000); $0.0108 per page details row ($10.8 per 1,000) and $0.00225 per comment or reply ($2.25 per 1,000) when those options are on; pages that are not found or not public are free |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | Seconds to a minute for 10 to 50 posts per page |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/facebook-page-posts-scraper if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/facebook-page-posts-scraper`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get posts

```bash
# A link to a single post, photo, video or reel in "pages" returns that post's comments
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~facebook-page-posts-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"pages":["https://www.facebook.com/NASA"],"maxPostsPerPage":10,"onlyPostsNewerThan":"30 days","includeSummary":true}'
```

The response is a JSON array: one object per post, plus optional summary, page and comment rows marked by `type`.

## Input fields

| Field | Meaning |
| --- | --- |
| `pages` (required) | Page links, page names or numeric ids. A link to one post, photo, video or reel gives that post's comments |
| `maxPostsPerPage` | Newest first, default 10. This is the cost cap: always set it |
| `onlyPostsNewerThan`, `onlyPostsOlderThan` | A date (`2026-11-01`) or a period (`7 days`). Skipped posts are free |
| `includeSummary` | Free row per page with averages, engagement by post type and best posts |
| `includePageDetails` | Row per page with bio, categories, contacts, address, rating and followers. Extra charge |
| `includeComments`, `maxCommentsPerPost` | Comments of each post, charged per comment |
| `includeCommentReplies` | Also the replies to comments |
| `includeTranscript` | Video captions as text, when the video has them. Free |
| `onlyNewPosts` | Only posts published since the last run on the same page |
| `maxPostsTotal` | One limit across all pages |

There are more fields on the Actor's page (https://apify.com/pomlabs/facebook-page-posts-scraper); these are the main ones.

## Output fields worth keeping

`url`, `text`, `date`, `reactionsCount` and each reaction type, `commentsCount`, `sharesCount`, photo and video links, page name and followers. Say how many posts were read when comparing performance: the newest 10 posts are not the whole history.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Page returned with a reason and no posts | The page does not exist, is not public or is blocked | Check the link; these rows are free |
| Fewer photos than the album has | Facebook sends at most 5 photos of an album to visitors | Use `photoCount` for the real number |

## Cost check before big batches

Cost is `pages x maxPostsPerPage x $0.002` at most, plus page details and comments if turned on. Above $5, tell the user the estimate and wait for a yes.

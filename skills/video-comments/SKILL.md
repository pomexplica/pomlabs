---
name: video-comments
description: Gets the public comments and replies of YouTube videos and Shorts as JSON, with likes, reply counts and author. Use when the user shares a video link and wants to know what viewers are saying, the most liked comments, recurring questions, objections or requests, sentiment, or quotes from the audience for research or content ideas. Runs on Apify with the user's own APIFY_TOKEN; pay per comment, no login to YouTube.
license: MIT
---

# Video comments (YouTube)

One Apify Actor by Pom Labs returns the comment section of public YouTube videos. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~youtube-comments-scraper` |
| Price | $0.001 per comment or reply delivered ($1 per 1,000) |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | 5 to 15 s for 100 comments |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/youtube-comments-scraper if they matter to the user.

## No terminal? Use the Apify connector

If the Apify connector (MCP server `mcp.apify.com`) is connected, skip the token setup and the `curl` call: call `call-actor` with `actor` set to the Actor name from the table (with `/` instead of `~`) and `input` set to the same JSON as below, then read the result with `get-dataset-items` and the `datasetId` it returns. The connector bills the user's own Apify account at the same prices.

## Setup (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get comments

```bash
# Watch URLs, youtu.be links, Shorts or the 11-character video id
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~youtube-comments-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"videos":["https://www.youtube.com/watch?v=jNQXAC9IVRw"],"maxCommentsPerVideo":100,"sortBy":"top","includeReplies":false}'
```

The response is a JSON array with one object per comment.

## Input fields

| Field | Meaning |
| --- | --- |
| `videos` (required) | Array of video links or ids. Duplicates are removed |
| `maxCommentsPerVideo` | Comments plus replies per video, default 100. This is the cost cap: always set it |
| `sortBy` | `top` (default) or `newest`, as in the YouTube comment section |
| `includeReplies` | `true` (default) also returns replies, marked with `isReply` and `parentCommentId`. Use `false` for a first read of what viewers think |
| `maxRepliesPerComment` | Default 5. Lower it to get more distinct top-level comments for the same cost |

## Output fields worth keeping

`text`, `likes`, `replyCount`, `author`, `publishedTimeText` (as YouTube shows it, e.g. "6 years ago"), `publishedAtApprox`, `isReply`, `parentCommentId`, `heartedByCreator`, `videoId`. Dates are approximate because YouTube only shows relative time.

When summarizing, quote comments exactly and say how many comments were read: 100 top comments are not the whole audience.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Empty array for a video | Comments are turned off, or the video is private or removed | Say so for that video; charges are per comment delivered |

## Cost check before big batches

Cost is `videos x maxCommentsPerVideo x $0.001` at most. Above 5,000 comments ($5), tell the user the estimate and wait for a yes.

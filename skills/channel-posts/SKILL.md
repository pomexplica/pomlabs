---
name: channel-posts
description: Gets the posts of any public Telegram channel as JSON: text, date, views, reactions, media links, forwards, replies and polls, plus the exact subscriber count, bio and subscriber growth. No Telegram login. Use when the user gives a Telegram channel and wants its recent posts, what got the most views, posts with a keyword or in a date range, or the size of the channel. Runs on Apify with the user's own APIFY_TOKEN; pay per post delivered.
license: MIT
---

# Channel posts (Telegram)

One Apify Actor by Pom Labs reads the public web preview of Telegram channels and returns their posts and channel details. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~telegram-channel-scraper` |
| Price | $0.001 per post delivered ($1 per 1,000) and $0.001 per channel info row ($1 per 1,000); posts Telegram hides on the web, channels not found and blocked pages are free |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | Seconds for 20 posts; longer for whole histories |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/telegram-channel-scraper if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/telegram-channel-scraper`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

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
# channels: a username, @username or a t.me link
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~telegram-channel-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"channels":["durov"],"maxPostsPerChannel":20,"includeChannelInfo":true}'
```

The response is a JSON array: one object per post, plus one channel row per channel, marked by `type`.

## Input fields

| Field | Meaning |
| --- | --- |
| `channels` | Usernames, `@username` or `t.me` links, one per line |
| `maxPostsPerChannel` | Newest first, default 20; `0` = all posts the channel shows on the web. This is the cost cap: always set it |
| `dateFrom`, `dateTo` | Only posts in this UTC day range (`YYYY-MM-DD`) |
| `lastNDays` | Only posts of the last N days |
| `keyword` | Only posts with these words |
| `excludeKeywords` | Skip posts with any of these words |
| `mediaTypes` | Keep only posts with these media types |
| `includeChannelInfo` | One row per channel with title, bio, exact subscribers and growth (default on) |
| `profileOnly` | Only the channel row, no posts |

There are more fields on the Actor's page (https://apify.com/pomlabs/telegram-channel-scraper); these are the main ones.

## Output fields worth keeping

Posts: `url`, `postId`, `date`, `text`, `views`, `reactions`, `reactionsTotal`, `forwardedFrom`, `media`, `links`, `hashtags`. Channel row: `title`, `description`, `subscribers`, `verified`, and subscriber growth over 7 and 30 days.

Only public channels are read, and only what Telegram shows on its public web preview. Growth figures fill in over time, so a channel read for the first time may have none.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Channel returned with no posts | The channel is private, has no web preview or does not exist | Say so; these are free |

## Cost check before big batches

Cost is `channels x maxPostsPerChannel x $0.001` at most, plus $0.001 per channel row. Above 5,000 posts ($5), tell the user the estimate and wait for a yes.

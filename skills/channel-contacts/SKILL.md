---
name: channel-contacts
description: Gets public data of YouTube channels from a URL, @handle, channel id or a keyword search: name, subscribers, videos, total views, country, joined date, description, About links and the emails written in the description and links. No YouTube login or API key. Use when the user wants creator contact emails, a lead list of channels in a niche, or to compare channel sizes. Runs on Apify with the user's own APIFY_TOKEN; pay per channel and per email found.
license: MIT
---

# Channel contacts (YouTube)

One Apify Actor by Pom Labs returns public YouTube channel data and the business emails creators publish. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~youtube-channel-email-scraper` |
| Price | $0.001 per channel delivered ($1 per 1,000) and $0.004 per email found ($4 per 1,000) (each distinct address); channels not found and blocked requests are free. Video rows, when asked for, have their own price on the Actor's page |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | Seconds for tens of channels |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/youtube-channel-email-scraper if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/youtube-channel-email-scraper`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get channels

```bash
# channels: URLs, @handles or channel ids; or use searchQueries to find channels by keyword
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~youtube-channel-email-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"channels":["@MrBeast"],"onlyChannelsWithEmail":false}'
```

The response is a JSON array with one object per channel.

## Input fields

| Field | Meaning |
| --- | --- |
| `channels`, `urls` | Channel URLs, @handles, channel ids or video links, one per line |
| `searchQueries` | Find channels for each keyword (up to 20) and scrape them |
| `maxResultsPerQuery` | Channels taken from each keyword search (default 20). This is a cost cap when searching |
| `onlyChannelsWithEmail` | Skip (and do not charge) channels with no email |
| `scanLinkedWebsites` | Also open the channel's own website for emails |
| `validateEmails` | Check each email found (valid, invalid, risky, unknown) |
| `joinedDateFrom`, `joinedDateTo` | Only channels created in this range |
| `maxResults` | Also list this many videos of each channel (0 = none), charged per video row |
| `language`, `country` | YouTube page language and region |

There are more fields on the Actor's page (https://apify.com/pomlabs/youtube-channel-email-scraper); these are the main ones.

## Output fields worth keeping

`channelName`, `handle`, `channelUrl`, `subscriberCount`, `videoCount`, `viewCount`, `country`, `joinedDate`, `description`, `links`, `emails`, `emailSources`, `hasHiddenBusinessEmail`, `isVerified`.

Only emails the creator wrote publicly are returned. `hasHiddenBusinessEmail` means YouTube hides one more behind a sign-in button, which this Actor does not read. Remind the user that outreach rules (such as anti-spam laws) are theirs to follow.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Channel returned with a status and no data | The channel does not exist or the request was blocked | Check the link; these are free |

## Cost check before big batches

Cost is `channels x $0.001` plus `$0.004` per email found. Above $5, tell the user the estimate and wait for a yes.

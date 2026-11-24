---
name: video-transcripts
description: Gets the transcript of a public YouTube, Instagram, TikTok or Facebook video as text, with optional timestamps, subtitle files (SRT/VTT) and translation. Use when the user shares a video, Short or Reel link and wants it summarized, quoted, translated, turned into notes or captions, or when a batch of video URLs needs transcripts for research or RAG. Runs on Apify with the user's own APIFY_TOKEN; pay per transcript, no login to the video platforms.
license: MIT
---

# Video transcripts (YouTube, Instagram, TikTok, Facebook)

Two Apify Actors by Pom Labs return the spoken text of public videos as JSON. This skill is written and maintained by Pom Labs, the author of both Actors.

## Quick reference

| | YouTube | Instagram, TikTok, Facebook |
| --- | --- | --- |
| Actor | `pomlabs~youtube-transcript-scraper` | `pomlabs~social-video-transcript` |
| Source of text | The video's caption tracks | Platform captions when present, otherwise speech-to-text |
| Price | $0.003 per transcript | $0.01 per transcript (first minute) + $0.008 per extra minute of speech-to-text |
| Translation | +$0.002 per language | +$0.002 per language |
| Auth | `Authorization: Bearer $APIFY_TOKEN` | same |
| Typical time | 5 to 15 s per video | 10 to 60 s per video |

Failed or skipped videos are not charged. Prices are the ones on each Actor's page; check `https://apify.com/pomlabs` if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/youtube-transcript-scraper`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

1. The user needs an Apify account and an API token from https://console.apify.com/account/integrations.
2. Read the token from the environment. Never print it and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get transcripts

Pick the Actor by the link's domain. One call accepts many URLs; keep each call to 25 URLs or fewer so it finishes inside the 300 s sync limit.

```bash
# YouTube: watch, youtu.be, Shorts, embed and live links, or the 11-character id
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~youtube-transcript-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"urls":["https://www.youtube.com/watch?v=jNQXAC9IVRw"],"languages":["en"],"includeTimestamps":false}'
```

```bash
# Instagram Reels and video posts, TikTok videos, Facebook videos and Reels
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~social-video-transcript/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"urls":["https://www.tiktok.com/@scout2015/video/6718335390845095173"],"language":"auto","includeTimestamps":false,"maxDurationSeconds":600}'
```

The response is a JSON array with one object per URL, in input order.

## Input fields

| Field | Actor | Meaning |
| --- | --- | --- |
| `urls` (required) | both | Array of video links |
| `languages` | YouTube | Caption languages in order of preference, e.g. `["pt","en"]`. Manual captions win over auto-generated |
| `language` | social | `auto` (default) or a code such as `en`, `pt`, `es` to force the speech-to-text language |
| `translateTo` | both | Extra languages to return with the same timestamps, e.g. `["en"]` |
| `includeTimestamps` | both | `true` adds a `segments` array (`start`, `duration` or `end`, `text`). Leave `false` for summaries to save tokens |
| `subtitleFormat` | both | `srt` or `vtt` adds a ready subtitle file as text; default `none` |
| `maxDurationSeconds` | social | Longer videos are skipped and not charged. Set it before a batch so cost is bounded |
| `includeComments`, `maxComments` | social | Also return top-level comments ($0.0005 each) |

## Output fields worth keeping

`status` (`ok` or an error code), `title`, `transcript` (plain text) and `segments` when requested. YouTube adds `channelName`, `language`, `isAutoGenerated`, `availableLanguages` and `charCount`; the social Actor adds `platform`, `author`, `durationSeconds` and `source` (`speech_to_text` or the caption track). Always check `status` per item before using `transcript`.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Item with `status` other than `ok` | That video has no captions, is private, or was removed | Report it for that URL and continue with the others; it was not charged |
| Empty `transcript` on a music or silent video | Nothing was spoken | Say so; do not invent text |

## Cost check before big batches

Estimate before running more than 50 videos: YouTube is `videos x $0.003`; social is `videos x $0.01` plus `$0.008` per extra minute. Tell the user the estimate and wait for a yes.

---
name: photo-geolocation
description: Finds where a photo or video was taken, from a file or a link (Instagram, TikTok, YouTube, X, Facebook and more): best-guess place with coordinates and radius, alternatives, confidence, visual clues, GPS in the file, spoken language, plus a privacy report and a check of a claimed place. Use when the user asks where a picture or clip was taken, wants to check a claimed location, or wants to see what a photo gives away before posting it. Runs on Apify with the user's own APIFY_TOKEN; pay per photo or video analyzed.
license: MIT
---

# Photo and video geolocation

One Apify Actor by Pom Labs estimates where a photo or video was taken and reports what it reveals. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~photo-video-geolocation` |
| Price | $0.0225 per photo or video analyzed ($22.5 per 1,000); links that cannot be opened are free |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | Tens of seconds per item; videos take longer |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/photo-video-geolocation if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/photo-video-geolocation`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Locate a photo or video

```bash
# urls: image or video links, or public post links
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~photo-video-geolocation/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"urls":["https://upload.wikimedia.org/wikipedia/commons/0/0f/0123_RUA_AUGUSTA_Lisbon.jpg"],"claimedPlace":"Lisbon, Portugal"}'
```

The response is a JSON array with one object per photo or video.

## Input fields

| Field | Meaning |
| --- | --- |
| `urls` | Links to photos (JPG, PNG, WebP, HEIC), videos (MP4 and others) or public posts on social sites |
| `upload` | Upload one photo or video instead of a link (in the Apify console) |
| `claimedPlace` | Optional: where someone says it was taken. The result says `consistent`, `inconsistent` or `uncertain`, with the distance |
| `transcribeSpeech` | Listen to the first minute of a video for the spoken language (default on) |
| `videoFrames` | Frames of each video to examine, 1 to 3 |

There are more fields on the Actor's page (https://apify.com/pomlabs/photo-video-geolocation); these are the main ones.

## Output fields worth keeping

`status`, `confidence`, `bestGuess` (place, coordinates, radius), `candidates`, `clues`, `fileGps`, `visibleLeaks`, `privacy`, and the claim check when `claimedPlace` is set.

This is an estimate, not proof. On street photos of cities it usually gets the city; in small inland towns it usually gets the region; indoor shots carry few clues. Always give the confidence and the radius with the place, and never present a guess as certain. Do not use it to find where a private person lives.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Item returned as not opened | The link is private or not a photo or video | Ask for the file or another link; these are free |
| Low confidence | Few clues in the image | Say so; do not rephrase it as a finding |

## Cost check before big batches

Cost is `items x $0.0225`. Above $5 (about 220 items), tell the user the estimate and wait for a yes.

---
name: video-download
description: Downloads a public video as an MP4 file, or its audio as M4A, MP3 or WAV, from TikTok, Instagram, YouTube, X, Facebook, Threads, Reddit, Vimeo and other sites, and returns a stable file link with title, author, duration, resolution and view counts. Use when the user shares a video link and wants the file saved, wants only the audio, wants a clip between two timestamps, wants a version without sound, or needs the video file as input for editing, analysis or archiving. Runs on Apify with the user's own APIFY_TOKEN; pay per file, no login to the platforms.
license: MIT
---

# Video download

One Apify Actor by Pom Labs saves public videos as files and returns a link to each one. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~video-downloader` |
| Price | $0.005 per file up to 10 MB ($0.015 for YouTube); + $0.004 per extra 10 MB; + $0.01 per minute only when a video must be converted to H.264; + $0.012 per MB only when a site forces residential traffic |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Sites | TikTok, Instagram, YouTube, X, Facebook, Threads, Reddit, Kwai, Vimeo, Snapchat Spotlight, Bilibili, Pinterest and others; the Actor's page has the current list |
| Typical time | 5 to 15 s per video; YouTube takes 1 to 5 minutes per video |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/video-downloader if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to `pomlabs/video-downloader` and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Download videos

Private, deleted, login-only and unsupported links are not charged. Because YouTube is slow, send YouTube links one per call, and other platforms up to 10 per call.

```bash
# TikTok, Instagram, X, Facebook, Threads, Reddit, Vimeo and others
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~video-downloader/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"urls":["https://www.tiktok.com/@scout2015/video/6718335390845095173"],"quality":"1080"}'
```

```bash
# YouTube, one link per call; here only the audio, as MP3
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~video-downloader/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"urls":["https://www.youtube.com/watch?v=jNQXAC9IVRw"],"audio":"only","audioFormat":"mp3"}'
```

The response is a JSON array with one object per link. `fileUrl` is a signed link that opens without a token: download the file from it directly, and share it only with whoever should have the file.

## Input fields

| Field | Meaning |
| --- | --- |
| `urls` (required) | Array of public video links |
| `quality` | Best resolution up to this one: `best`, `2160`, `1440`, `1080` (default), `720`, `480`, `360`, `240`, `144`, `smallest` |
| `audio` | `video` (default, with sound), `none` (muted MP4) or `only` (just the sound) |
| `audioFormat` | With `audio: only`: `m4a` (default, the original track), `mp3` or `wav` |
| `clipStart`, `clipEnd` | Seconds or `mm:ss`. Downloads only that part. Works best on YouTube and X |
| `compatibleMp4` | `true` (default) prefers H.264 with AAC, which plays everywhere; conversion is charged per minute only when the platform has no H.264 version |
| `maxFileSizeMb` | Default 500. Bigger files are skipped and not charged. Lower it to bound the cost |

## Output fields worth keeping

`status` (`ok` or an error code), `platform`, `title`, `author`, `durationSeconds`, `uploadDate`, `viewCount`, `likeCount`, `commentCount`, `fileUrl`, `fileKey`, `fileSizeBytes`, `width`, `height`, `videoCodec`, `audioCodec`, `hasAudio`, `sha256`.

The file lives in the run's key-value store in the user's Apify account and follows that account's retention: tell the user to save it if they need it for long. The user is responsible for having the right to download and reuse the video.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Usually a YouTube link in a batch | Send that link alone and retry once |
| `run-failed` with status `TIMED-OUT` | The run did not finish inside the timeout, seen with YouTube mixed with other links | Send YouTube links one per call |
| Item with `status` other than `ok` | The video is private, deleted, needs login or the site is not supported | Report the item's `error` text for that link; it was not charged |

## Cost check before big batches

A short clip costs $0.005 ($0.015 on YouTube). A long video costs more: about `$0.004 x (size in MB / 10)` on top. Above 50 videos, or for any video longer than 30 minutes, tell the user the estimate and wait for a yes.

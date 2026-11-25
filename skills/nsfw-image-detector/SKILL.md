---
name: nsfw-image-detector
description: Checks image URLs for nudity, sexual, violent, gory, hateful, drug, weapon and self-harm content and returns as JSON 14 scores, an allow / review / block verdict with reasons, plus the image category and content tags in the same call. Use when the user needs to moderate user-uploaded images, filter NSFW pictures from a list, check a batch of product or profile photos before publishing, or tag images by content. Runs on Apify with the user's own APIFY_TOKEN; pay per image.
license: MIT
---

# NSFW image detector and tagger

One Apify Actor by Pom Labs scores public images for unsafe content and tags them. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~image-moderation-tagger` |
| Price | $0.002 per image analyzed ($2 per 1,000); tagging included |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | 2 to 5 s for a few images (median 3 s in our runs) |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/image-moderation-tagger if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/image-moderation-tagger`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Check images

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~image-moderation-tagger/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"imageUrls":["https://upload.wikimedia.org/wikipedia/commons/3/3a/Cat03.jpg"],"tagging":true}'
```

The response is a JSON array with one object per image.

## Input fields

| Field | Meaning |
| --- | --- |
| `imageUrls` (required) | Public http(s) links to JPEG, PNG, WebP, GIF, BMP or TIFF (up to 20 MB), or `data:` URIs |
| `tagging` | On by default: category, up to 40 tags and the kind of image, at the same price |
| `customLabels` | Up to 10 plain-English descriptions scored 0 to 1 on every image ("a person wearing a helmet") |
| `blockThreshold`, `reviewThreshold` | Defaults 0.6 and 0.35 |
| `blockCategories`, `reviewCategories` | Which categories block and which only flag for review |
| `allowArtisticNudity`, `allowBreastfeeding` | Context exceptions (off and on by default) |
| `maxFrames` | Frames checked in animated GIF or WebP (default 3) |

## Output fields worth keeping

`imageUrl`, `verdict` (`allow`, `review`, `block`), `reasons`, `nsfw`, `nudityLevel`, `nudityContext`, `scores` (one per category), `topCategory`, `topScore`, `category`, `tags`, `imageKind`, `hasText`, `hasFace`, `width`, `height`, `sha256`, `duplicateOf`.

The verdict is a score against thresholds, not a legal judgment. For borderline cases (`review`), say so and leave the decision to a person.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| An item with `status` `error` | The image could not be downloaded or decoded | Report that URL; it is not charged |

## Cost check before big batches

Cost is `images x $0.002`. Above 2,500 images ($5), tell the user the estimate and wait for a yes.

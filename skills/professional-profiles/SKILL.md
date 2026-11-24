---
name: professional-profiles
description: Gets the public LinkedIn profile of people as JSON - headline, current position and company, location, about, work experience with dates, education, languages, follower count and the similar profiles LinkedIn shows. Use when the user gives a linkedin.com/in link or a LinkedIn username and wants a person's background, current role or career history, wants to enrich a list of leads, candidates or speakers, or wants similar people to a given profile. Runs on Apify with the user's own APIFY_TOKEN; pay per profile, no LinkedIn login or cookies.
license: MIT
---

# Professional profiles (LinkedIn)

One Apify Actor by Pom Labs returns what a logged-out visitor sees on a LinkedIn profile page. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~linkedin-profile-scraper` |
| Price | $0.002 per profile ($2 per 1,000). Profiles not found or refused are not charged |
| Auth | `Authorization: Bearer $APIFY_TOKEN`. No LinkedIn account or cookie is used |
| Typical time | 3 to 10 s per profile |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/linkedin-profile-scraper if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to `pomlabs/linkedin-profile-scraper` and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get profiles

The input is the profile URL or the username (the part after `/in/`). A person's name alone is not enough: if the user gives only a name, find the LinkedIn URL first (web search) and confirm it is the right person. Keep each call to 25 profiles or fewer.

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~linkedin-profile-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"profiles":["https://www.linkedin.com/in/satyanadella"],"includeSimilarProfiles":true}'
```

The response is a JSON array with one object per profile.

## Input fields

| Field | Meaning |
| --- | --- |
| `profiles` | Array of profile URLs (`linkedin.com/in/...`) or usernames |
| `includeSimilarProfiles` | `true` (default) adds the similar profiles LinkedIn shows next to each one. Set `false` for plain enrichment to save tokens |

## Output fields worth keeping

`status` (`ok` or an error code), `name`, `headline`, `location`, `countryCode`, `about`, `currentPosition`, `currentCompany`, `followers`, `connections` (LinkedIn's text, e.g. `500+`), `linkedinUrl`, `experience` (each with `title`, `company`, `companyUrl`, `startDate`, `endDate`, `duration`, `location`), `education` (`school`, `degree`, `startDate`, `endDate`), `languages`, `websites`, `similarProfiles` (`name`, `headline`, `location`, `url`).

`fullExperienceVisible: false` means LinkedIn showed only part of the work history to logged-out visitors: say so instead of presenting the list as complete. Only what the person made public is returned; there are no emails or phone numbers.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Item with `status` other than `ok` | No profile under that username, or LinkedIn refused to show it without login | Check the username against the real URL and report the item's `error` text; profiles not found are not charged |
| A different person than expected | The username was guessed from a name | Find the exact profile URL and run again |

## Cost check before big batches

Cost is `profiles x $0.002`. Above 2,500 profiles ($5), tell the user the estimate and wait for a yes.

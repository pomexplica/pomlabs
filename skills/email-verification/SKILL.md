---
name: email-verification
description: Verifies email addresses without sending a message: syntax, mail server, a real mailbox check, catch-all domains, disposable and role addresses, provider and typo fixes (Yahoo and Outlook by account lookup). Use when the user has a list of emails to clean, wants to know if an address exists before outreach, or wants to cut bounces. Runs on Apify with the user's own APIFY_TOKEN; pay per conclusive result.
license: MIT
---

# Email verification

One Apify Actor by Pom Labs checks email addresses in bulk and returns a status for each one, without sending any email. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~email-verifier` |
| Price | $0.00085 per email with a conclusive result ($0.85 per 1,000); `unknown` results and syntax errors are free |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | Seconds for a short list; large lists take minutes (one connection at a time per mail provider) |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/email-verifier if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/email-verifier`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Verify emails

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~email-verifier/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"emails":["jane.doe@yahoo.com","someone@outlook.com","sales@example.com"],"greylistRetrySeconds":60}'
```

The response is a JSON array with one object per address.

## Input fields

| Field | Meaning |
| --- | --- |
| `emails` (required) | Addresses, one per line. Commas, `mailto:` and `<brackets>` are cleaned; duplicates are checked once |
| `smtpCheck` | Check the mailbox on the mail server (default `true`). `false` gives only syntax, MX, disposable and role checks |
| `checkCatchAll` | Detect catch-all domains (default `true`) |
| `greylistRetrySeconds` | Retry greylisted addresses once after this wait (default 60; 0 = no retry) |
| `timeoutSeconds` | Wait per mail server answer (default 12) |
| `inputDatasetId` | An Apify dataset id whose emails join the list (for example the output of a lead scraper) |
| `emailField` | Field of each dataset item that holds the email (default `email`) |
| `skipInvalidSyntax` | Leave entries that are not emails out of the results |
| `concurrency` | Mail domains checked at the same time, 1 to 16 (default 8) |

There are more fields on the Actor's page (https://apify.com/pomlabs/email-verifier); these are the main ones.

## Output fields worth keeping

`email`, `status` (`valid`, `invalid`, `catch_all`, `disposable`, `role` or `unknown`), `reason`, `reasonText`, `provider`, `isDisposable`, `isRole`, `isCatchAll`, `mxFound`, `smtpCode`, `didYouMean`, `score`.

Explain the statuses to the user: `catch_all` means the domain accepts any address, so the mailbox cannot be confirmed; `unknown` means the server did not answer (not charged) and the address can be retried later. Do not call a `catch_all` or `unknown` address valid.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| Many `unknown` results | Mail servers that greylist, time out or block checks | Retry those later; they are free |
| Every address of one domain is `catch_all` | The domain accepts anything | Report it as unconfirmed, not valid |

## Cost check before big batches

Cost is `emails x $0.00085` at most. Above $5 (about 5,900 addresses), tell the user the estimate and wait for a yes.

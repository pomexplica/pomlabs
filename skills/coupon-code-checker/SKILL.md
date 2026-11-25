---
name: coupon-code-checker
description: Checks whether Mercado Livre or Amazon Brazil coupon codes really work - valid, expired, invalid or not applicable - and returns as JSON the discount, minimum purchase, maximum discount, expiry date and the eligible products link (Mercado Livre), checked live on a real account. Use when the user has promo codes for Mercado Livre or Amazon.com.br and wants to know which ones work before sharing or using them, or wants to validate a coupon list for a deals site. Runs on Apify with the user's own APIFY_TOKEN; pay per code.
license: MIT
---

# Coupon code checker (Mercado Livre, Amazon Brazil)

One Apify Actor by Pom Labs tries each code on a real store account and reports what the store answers. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~mercadolivre-coupon-checker` |
| Price | $0.01 per code checked |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | 5 to 15 s per code (26 to 89 s per run of 3 codes in our runs); up to 40 codes per hour per store |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/mercadolivre-coupon-checker if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/mercadolivre-coupon-checker`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Check codes

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~mercadolivre-coupon-checker/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"store":"mercadolivre","codes":["MELIACHA30","NAOEXISTE123"],"maxCodes":20}'
```

The response is a JSON array with one object per code.

## Input fields

| Field | Meaning |
| --- | --- |
| `store` | `mercadolivre` (default) or `amazon` (Amazon.com.br) |
| `codes` (required) | Codes, letters and digits, 3 to 30 characters; case and spaces do not matter |
| `maxCodes` | Safety limit per run, default 20 |

## Output fields worth keeping

`code`, `status` (`valid`, `expired`, `invalid`, `exists_not_applicable`; charged only for these four), `valid`, `message`, `storeMessage` (the store's own words, in Portuguese), `discount`, `discountPercent`, `appliesTo`, `minimumPurchaseBRL`, `maxDiscountBRL`, `expiresAt`, `eligibleProductsUrl`, `checkedAt`.

A code that works on the checking account can still be limited to some buyers (first purchase, selected products); quote `storeMessage` and `appliesTo` instead of promising the discount.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |
| `status` `rate_limited` | The hourly or daily check limit of the store account was reached | Not charged; tell the user and try later |
| `status` `validator_offline` or `invalid_input` | The checking account is offline, or the code is not 3 to 30 letters and digits | Not charged; fix the code or try later |

## Cost check before big batches

Cost is `codes x $0.01`, and runs go at a human pace. More than 20 codes: split into runs and tell the user it takes about 15 s per code.

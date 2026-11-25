---
name: ai-answer-visibility
description: Asks ChatGPT, Claude and Perplexity, with live web search, the questions buyers ask, and returns as JSON whether a brand is mentioned, in what position among competitors, which competitors appear and which sources the AI cites. Use when the user wants to know if AI assistants recommend their brand or product (GEO, AI search visibility, LLM SEO), how they compare with competitors in AI answers, or which pages the AI relies on. Runs on Apify with the user's own APIFY_TOKEN; pay per answer.
license: MIT
---

# AI answer visibility (ChatGPT, Claude, Perplexity)

One Apify Actor by Pom Labs asks AI assistants with live web search the questions a buyer would type, and measures the brand in each answer. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~ai-brand-visibility` |
| Price | $0.03 per answer delivered; $0.01 per answer when the user brings their own OpenAI or Perplexity key |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | 5 to 30 s per run of a few questions (median 20 s in our runs) |

One answer = one question asked to one AI. Prices are the ones on the Actor's page; check https://apify.com/pomlabs/ai-brand-visibility if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/ai-brand-visibility`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Check a brand

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~ai-brand-visibility/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"prompts":["What is the best CRM for a small business?"],"brand":"HubSpot","brandDomain":"hubspot.com","competitors":["Salesforce","Pipedrive","Zoho"],"engines":["claude","chatgpt","perplexity"]}'
```

The response is a JSON array with one object per question and AI.

## Input fields

| Field | Meaning |
| --- | --- |
| `prompts` | The questions, in any language. Each one is asked to each engine |
| `category` | If `prompts` is empty, 3 standard questions are built from it ("What is the best <category>?") |
| `brand` | Name to look for (case-insensitive, whole word) |
| `brandAliases` | Other spellings or product names that count as the brand |
| `brandDomain` | Optional: checks whether the AI cited or found a page from this site |
| `competitors` | Names to track; the output ranks the brand among them by order of first mention |
| `engines` | Any of `claude` (default), `chatgpt`, `perplexity` |
| `openaiApiKey`, `perplexityApiKey`, `openaiModel` | Optional own keys: the user pays the AI provider and the Actor charges the lower price |

## Output fields worth keeping

`prompt`, `engine`, `model`, `brandMentioned`, `brandMentions`, `brandRankAmongTracked`, `competitorsMentioned`, `brandDomainCited`, `brandDomainCitedUrls`, `citedSources`, `answer`, `checkedAt`.

AI answers change from one run to the next. When reporting, say how many questions and engines were asked, and prefer several questions over one before concluding that a brand "is" or "is not" recommended.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Split into smaller batches and retry once |

## Cost check before big batches

Cost is `prompts x engines x $0.03`. Above 100 answers ($3), tell the user the estimate and wait for a yes.

---
name: job-listings
description: Gets job listings as JSON from Indeed (62 country sites) and Glassdoor by keyword and location, job link or job key - title, company, salary with yearly min and max, posted date, remote or hybrid, job type, apply link and the full description - plus optional company contacts, company details and a free salary summary per search (p25, median, p75). Use when the user wants open jobs for a role or city, salary ranges for a job, a list of companies hiring, new postings to monitor, or job data for a spreadsheet, CRM or job board. Runs on Apify with the user's own APIFY_TOKEN; pay per job delivered.
license: MIT
---

# Job listings (Indeed and Glassdoor)

One Apify Actor by Pom Labs reads Indeed and Glassdoor job searches. This skill is written and maintained by Pom Labs, the author of the Actor.

## Quick reference

| | |
| --- | --- |
| Actor | `pomlabs~indeed-jobs-scraper` |
| Price | $0.0001 per job delivered ($0.10 per 1,000) on the free Apify plan, less on paid plans; extras: company contacts $0.045 per job with an e-mail found, search page signals $0.0009 per job, company details $0.0009 per job found; searches with no jobs are free; no start fee |
| Auth | `Authorization: Bearer $APIFY_TOKEN` |
| Typical time | 6 s for 10 jobs, 8 s for 10 jobs from Indeed and Glassdoor with the summary (our runs, 27/11/2026) |

Prices are the ones on the Actor's page; check https://apify.com/pomlabs/indeed-jobs-scraper if they matter to the user.

## Two ways to run it

**1. Apify connector (use this when it is there).** If the Apify tools `call-actor` and `get-dataset-items` are available (the plugin version of this skill declares the connector `https://mcp.apify.com`, and Apify is also in Claude's connector directory), call `call-actor` with `actor` set to the Actor name from the table, written with `/` instead of `~` (for example `pomlabs/indeed-jobs-scraper`), and `input` set to the same JSON shown in the examples below. `call-actor` waits up to 45 s (`waitSecs`); if it answers with `status` `RUNNING`, call `get-actor-run` with the `runId` until the status is `SUCCEEDED`. Then call `get-dataset-items` with the dataset id found at `storages.datasets.default.id`. To cap what a run can cost, pass `callOptions` with `maxTotalChargeUsd`. The user signs in to Apify once in the browser, and no token is handled here.

**2. API token and `curl`.** When those tools are not available, use the setup and the `curl` calls below.

Both ways bill the user's own Apify account at the prices in the table.

## Setup for the token path (once)

The user needs an Apify account and an API token from https://console.apify.com/account/integrations, exported as `APIFY_TOKEN`. Never print the token and never put it in a URL.

```bash
test -n "$APIFY_TOKEN" || echo "APIFY_TOKEN is not set: ask the user to create one at https://console.apify.com/account/integrations"
```

If the token is missing, stop and tell the user exactly that. Do not fall back to scraping the page yourself.

## Get jobs

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~indeed-jobs-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"keywords":["python developer"],"location":"New York, NY","maxItemsPerKeyword":10}'
```

Indeed and Glassdoor together, with the free salary summary:

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/pomlabs~indeed-jobs-scraper/run-sync-get-dataset-items?timeout=280" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"keywords":["data analyst"],"location":"Chicago, IL","sources":["indeed","glassdoor"],"maxItemsPerKeyword":5,"includeSummary":true}'
```

The response is a JSON array: one object per job (`source` is `indeed` or `glassdoor`) and, with `includeSummary`, one object per search with `type` `summary`.

## Input fields

| Field | Meaning |
| --- | --- |
| `keywords` | Job titles or keywords; each one is a separate search |
| `startUrls` | Indeed search links (any filter set on the site) or job links (`viewjob?jk=...`) |
| `jobKeys` | Indeed job keys (the 16 characters after `jk=`) or job links, for jobs you already know |
| `location` | City, state, postcode or region as typed on Indeed (`New York, NY`, `London`, `São Paulo, SP`); empty = whole country |
| `country` | Indeed site: `US` (default) = indeed.com, `BR` = br.indeed.com, `GB` = uk.indeed.com, and so on |
| `sources` | `["indeed"]` (default), `["glassdoor"]` or both; with both, the maximum applies to each site and a job on both comes once |
| `maxItemsPerKeyword` | Jobs per keyword (default 50) |
| `postedWithinDays` | Only jobs posted in the last N days (use this for fresh jobs; `sort: "date"` is not strictly by date) |
| `jobType`, `experienceLevel`, `workplace`, `remoteOnly` | Indeed filters: job type, level, remote or hybrid |
| `minSalary`, `onlyWithSalary` | Indeed salary filter (per year, country currency); drop jobs with no salary |
| `postedBy`, `applyType` | Employer-posted only; Indeed Apply only or external apply only |
| `splitSearches` | On by default: when Indeed ends a search early (a few hundred jobs), the search is repeated with other filters to reach the maximum |
| `includeDescription` | Full description as plain text (on by default, no extra cost) |
| `uniqueCompanies` | At most one job per employer: a list of companies hiring |
| `strictMatch` | Drop jobs whose title and description do not contain every word of the keyword |
| `resumeKeywords` | Your skills; each job gets `resumeMatchScore` (0-100) and the matched and missing keywords |
| `includeContacts` | E-mails, phones and careers page from each company's own website (extra price above) |
| `includeCompanyDetails` | Founding year, revenue, type, headquarters and size from Glassdoor (extra price above) |
| `includeHiringSignals` | Sponsored flag and position, hires needed and other search page signals (extra price above) |
| `includeSummary` | One free row per search: salary p25, median and p75 per year by currency, remote, hybrid and easy apply shares, levels, top companies, places, benefits and skills |
| `onlyNewJobs`, `monitorName` | Skip jobs already delivered in earlier runs with the same monitor name |

## Output fields worth keeping

Jobs: `title`, `company`, `location`, `city`, `state`, `country`, `datePosted`, `salary` (`min`, `max`, `period`, `currency`, `isEstimated`, `yearlyMin`, `yearlyMax`, `yearlyMid`), `salaryText`, `jobTypes`, `isRemote`, `isHybrid`, `isEasyApply`, `experienceLevel`, `benefits`, `skillsAndRequirements`, `description`, `url`, `applyUrl`, `companyRating`, `companyReviewCount`, `companySize`, `companyIndustry`, `companyWebsite`, `source`. Glassdoor jobs also bring `skills`, `yearsOfExperience`, `isSponsored` and `companyDetails`.

Summary rows (`type: "summary"`): `search`, `jobs`, `salaryPerYear` (per currency: `jobsWithSalary`, `p25`, `median`, `p75`), `remotePercent`, `hybridPercent`, `easyApplyPercent`, `experienceLevels`, `jobTypes`, `topCompanies`, `topLocations`, `topBenefits`, `topSkillsAndRequirements`, `newestPost`.

Salary can be missing on a job: not every posting shows one. Say so when summarizing pay, and prefer the summary row, which counts only jobs with a salary.

## When something fails

| What you see | Meaning | What to do |
| --- | --- | --- |
| HTTP 401 | Token missing or wrong | Ask the user for a valid `APIFY_TOKEN` |
| HTTP 402 or a message about usage limits | The Apify account is out of credit | Tell the user; do not retry |
| HTTP 408, or the call runs past 280 s | Batch too large for a sync call | Fewer keywords or a smaller `maxItemsPerKeyword`, and retry once |
| A search returns no jobs | No match for that keyword and place (it is free) | Widen the location or remove filters |

## Cost check before big batches

Cost is about `jobs x $0.0001`, plus the extras that are on. `includeContacts` is the expensive one ($0.045 per job with an e-mail found): above 100 jobs with contacts ($4.50), tell the user the estimate and wait for a yes.

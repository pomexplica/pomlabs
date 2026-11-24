# Privacy

Last updated: 24 November 2026.

This repository holds agent skills (text instructions) and a plugin manifest published by Pom Labs. It contains no program that runs on its own.

## What the skills and the plugin collect

Nothing. They have no telemetry, no analytics and no server of their own. Pom Labs receives no data from your agent, your prompts or your files.

## What happens when a skill is used

Your agent starts a Pom Labs Actor on Apify, through the Apify connector (`mcp.apify.com`) or the Apify API (`api.apify.com`), with your own Apify account. What is sent is the input shown in each `SKILL.md`: the links, usernames, company pages, subreddits or search words you asked about, and the options of the run.

- The run, its input and its results are stored in your Apify account, under Apify's retention rules and privacy policy: https://apify.com/privacy-policy
- As the author of the Actors, Pom Labs sees through Apify the aggregate usage and billing figures of each Actor. Pom Labs does not receive your Apify token.
- The Actors read public pages only. Results can include personal data that people made public, such as the author name of a comment or the owner of a profile. You decide what to do with it, and the data protection law that applies to you applies to that use.

## Contact

Open an issue at https://github.com/pomexplica/pomlabs/issues.

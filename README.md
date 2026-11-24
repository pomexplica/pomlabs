# Pom Labs skills

[![skills.sh](https://skills.sh/b/pomexplica/pomlabs)](https://skills.sh/pomexplica/pomlabs)

Agent skills that give Claude, Codex, Cursor and other AI agents data from public social and business pages: video transcripts and downloads, comments, creator and professional profiles, company pages, community threads, short posts and marketplace products. Each skill teaches the agent to call one of the [Pom Labs Actors on Apify](https://apify.com/pomlabs) with your own Apify account. No login to the platforms themselves, and you pay only for what is returned.

Written and maintained by Pom Labs, the author of the Actors.

## Not a developer? Paste this into your agent

> Install the skills from https://github.com/pomexplica/pomlabs and set everything up for me. Tell me where to create my Apify API token and where to put it, then test it by getting the transcript of https://www.youtube.com/watch?v=jNQXAC9IVRw

On claude.ai, connect the **Apify** connector (Settings, Connectors) and the skills use it: you sign in to Apify in the browser and no token is needed.

## Skills

| Skill | Ask your agent | Data from | Price |
| --- | --- | --- | --- |
| [`video-transcripts`](skills/video-transcripts/SKILL.md) | "Summarize this video: `<link>`" · "Turn this Reel into an SRT file" · "Translate what is said in this TikTok to English" | YouTube, Instagram, TikTok, Facebook | $0.003 per YouTube transcript; $0.01 on the other platforms |
| [`video-comments`](skills/video-comments/SKILL.md) | "What are people asking in the comments of this video?" · "Give me the 20 most liked comments" | YouTube | $0.001 per comment |
| [`creator-profile-stats`](skills/creator-profile-stats/SKILL.md) | "How many followers does @nasa have, and how do its Reels usually do?" · "Which of this creator's last Reels beat their normal?" | Instagram | $0.0025 per profile; + $0.003 with Reels stats |
| [`company-pages`](skills/company-pages/SKILL.md) | "Enrich this list of companies with size, industry and website" · "What has this company been posting?" | LinkedIn | $0.003 per company |
| [`community-research`](skills/community-research/SKILL.md) | "What do people on Reddit complain about in `<product>`?" · "Get this year's threads about proxies from r/webscraping" | Reddit | $0.002 per post or comment |
| [`professional-profiles`](skills/professional-profiles/SKILL.md) | "What is this person's background? `<linkedin.com/in/...>`" · "Enrich these speakers with current role and company" | LinkedIn | $0.002 per profile |
| [`short-post-search`](skills/short-post-search/SKILL.md) | "What are people posting on Threads about `<topic>`?" · "Get the replies to this Threads post" | Threads | $0.002 per post or profile |
| [`marketplace-products`](skills/marketplace-products/SKILL.md) | "Find the best-selling wireless earbuds on AliExpress with prices" · "Compare sellers for this product in Brazil" | AliExpress | $0.001 per product |
| [`video-download`](skills/video-download/SKILL.md) | "Save this video as MP4: `<link>`" · "Give me only the audio of this video as MP3" | TikTok, Instagram, YouTube, X, Facebook and others | $0.005 per file ($0.015 on YouTube) |

Prices are per result and are the ones shown on each Actor's page at https://apify.com/pomlabs, where the current values live. Apify's free plan comes with monthly platform credit ($5 at the time of writing, see https://apify.com/pricing), enough for about 1,600 YouTube transcripts.

## Install

Any agent (Claude Code, Codex, Cursor, Gemini CLI and others), with the [skills CLI](https://skills.sh):

```bash
npx skills add pomexplica/pomlabs
```

One skill only:

```bash
npx skills add pomexplica/pomlabs --skill video-transcripts
```

Claude Code, as a plugin:

```
/plugin marketplace add pomexplica/pomlabs
/plugin install pomlabs-social-data@pomlabs
```

Manual: copy a folder from `skills/` into your agent's skills directory (for Claude Code, `~/.claude/skills/`).

## Setup

You need an Apify account (the free plan is enough to start). Then one of:

- **Apify connector.** Installed as a plugin, this repository declares the Apify MCP server (`https://mcp.apify.com`, see [`.mcp.json`](.mcp.json)). Your agent asks you to sign in to Apify in the browser the first time. On claude.ai, add the Apify connector.
- **API token.** Copy your token from https://console.apify.com/account/integrations and make it available to your agent as the environment variable `APIFY_TOKEN`. The skills never print it and never place it in a URL.

## What the skills do and do not do

- They run Pom Labs Actors through the Apify connector (`mcp.apify.com`) or the Apify API (`api.apify.com`), and contact nothing else. There is no telemetry and no instruction fetched from anywhere at run time: what you read in each `SKILL.md` is all there is.
- They read public pages only. Private accounts, private videos and anything behind a login are out of reach.
- Before a large batch, the agent estimates the cost and asks you first.

Trigger tests for each skill (requests that should and should not load it) are in [`tests/triggers.md`](tests/triggers.md).

## Privacy

The plugin and its skills collect nothing and store nothing. See [PRIVACY.md](PRIVACY.md).

## Trademarks

YouTube, Instagram, Facebook, Threads, TikTok, LinkedIn, Reddit, X and AliExpress are trademarks of their owners. These skills work with public data from those platforms and are not affiliated with or endorsed by them.

## License

MIT. See [LICENSE](LICENSE).

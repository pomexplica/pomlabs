# Pom Labs skills

[![skills.sh](https://skills.sh/b/pomexplica/pomlabs)](https://skills.sh/pomexplica/pomlabs)

Agent skills that give Claude, Codex, Cursor and other AI agents data from public social and business pages: video transcripts and downloads, comments, creator and professional profiles, company pages, community threads, short posts, marketplace products, search trends, brand visibility in AI answers, past versions of web pages, image moderation and coupon checks. Each skill teaches the agent to call one of the [Pom Labs Actors on Apify](https://apify.com/pomlabs) with your own Apify account. No login to the platforms themselves, and you pay only for what is returned.

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
| [`search-trends`](skills/search-trends/SKILL.md) | "Is interest in air fryers growing in the US?" · "What are the rising searches related to `<topic>`?" | Google Trends | $0.002 per keyword |
| [`ai-answer-visibility`](skills/ai-answer-visibility/SKILL.md) | "Does ChatGPT recommend our product when people ask for the best `<category>`?" · "Which competitors do AI assistants mention instead of us?" | ChatGPT, Claude, Perplexity | $0.03 per answer ($0.01 with your own key) |
| [`page-history`](skills/page-history/SKILL.md) | "What did this pricing page say a year ago?" · "When did this competitor change its terms?" | Wayback Machine, Arquivo.pt, archive.today, Common Crawl | $0.0025 per version |
| [`nsfw-image-detector`](skills/nsfw-image-detector/SKILL.md) | "Check these uploaded images for nudity or violence" · "Tag these product photos by content" | Any public image URL | $0.002 per image |
| [`coupon-code-checker`](skills/coupon-code-checker/SKILL.md) | "Which of these Mercado Livre coupon codes still work?" · "Is this Amazon Brazil promo code valid?" | Mercado Livre, Amazon Brazil | $0.01 per code |
| [`product-reviews`](skills/product-reviews/SKILL.md) | "What do buyers complain about in this Amazon product?" · "Get the 1-star reviews of `<ASIN>`" | Amazon | $0.0015 per review |
| [`sold-listings`](skills/sold-listings/SKILL.md) | "What did this item actually sell for on eBay?" · "Sold prices of used iPhone 13 in the last 90 days" | eBay | $0.002 per sold listing |
| [`email-verification`](skills/email-verification/SKILL.md) | "Which of these emails are deliverable?" · "Clean this lead list before sending" | Any email address | $0.00085 per conclusive email |
| [`page-posts`](skills/page-posts/SKILL.md) | "What has this Facebook page posted this month?" · "Get the comments of this page's last posts" | Facebook pages | $0.002 per post |
| [`social-search`](skills/social-search/SKILL.md) | "What are people posting on Facebook about `<topic>`?" · "Find buy-and-sell listings for bikes in my city" | Facebook search, groups, Marketplace | $0.001 per post, profile or page |
| [`ad-library`](skills/ad-library/SKILL.md) | "Which ads is this competitor running on Google?" · "Show the YouTube ads of `<advertiser>`" | Google Ads Transparency Center | $0.0015 per ad |
| [`business-leads`](skills/business-leads/SKILL.md) | "List dentists in Curitiba with phone and website" · "Find the emails of these local businesses" | Google Maps | $0.0036 per place |
| [`post-comments`](skills/post-comments/SKILL.md) | "What are people saying under this Instagram post?" · "Get the replies to this Reel" | Instagram | $0.001 per comment |
| [`web-search`](skills/web-search/SKILL.md) | "Search Google, Bing and DuckDuckGo for `<query>` and compare" · "Read the top results as Markdown" | Google, Bing, DuckDuckGo, Brave and others | $0.0025 per engine answered |
| [`photo-geolocation`](skills/photo-geolocation/SKILL.md) | "Where was this photo taken?" · "Estimate the location of this video" | Any public photo or video URL | $0.0225 per photo or video |
| [`marketplace-search-br`](skills/marketplace-search-br/SKILL.md) | "Find the cheapest air fryer on Shopee Brazil" · "Get the reviews of this Shopee product" | Shopee Brazil | $0.0003 per product |
| [`channel-posts`](skills/channel-posts/SKILL.md) | "What did this Telegram channel post this week?" · "Search a public channel for `<word>`" | Telegram public channels | $0.001 per post |
| [`company-reviews`](skills/company-reviews/SKILL.md) | "What do customers say about `<company>` on Trustpilot?" · "Get this year's 1-star reviews" | Trustpilot | $0.0005 per review |
| [`channel-contacts`](skills/channel-contacts/SKILL.md) | "Find the business email of these YouTube channels" · "List channels about `<topic>` with contacts" | YouTube | $0.001 per channel; $0.004 per email |
| [`job-listings`](skills/job-listings/SKILL.md) | "Find remote Python developer jobs in New York with salaries" · "What do data analysts earn in Chicago, by Indeed and Glassdoor?" | Indeed (62 countries), Glassdoor | $0.0001 per job |

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

Gemini CLI, as an extension (the skills plus the `pomlabs` MCP server with every tool; it asks for your Apify API token):

```bash
gemini extensions install https://github.com/pomexplica/pomlabs
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

YouTube, Instagram, Facebook, Threads, TikTok, LinkedIn, Reddit, X, AliExpress, Google, ChatGPT, Claude, Perplexity, Mercado Livre and Amazon are trademarks of their owners. These skills work with public data from those platforms and are not affiliated with or endorsed by them.

## License

MIT. See [LICENSE](LICENSE).

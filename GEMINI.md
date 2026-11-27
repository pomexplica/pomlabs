# Pom Labs data tools

The `pomlabs` MCP server runs Pom Labs Actors on Apify with the user's Apify token. Each tool is one Actor (YouTube and social video transcripts, YouTube comments, Instagram profile data, LinkedIn company and profile, Reddit posts and comments, Threads, AliExpress, Google Trends, AI brand visibility, Google Maps business leads, Google Ads transparency, multi-engine web search, photo/video geolocation, Telegram channels, Amazon reviews, email verifier, Trustpilot reviews, YouTube channel e-mails, image moderation, coupon checks, page history, video download).

- Call the tool with the smallest input that answers the question (one URL, a small `maxItems`), then grow.
- A run can take from a few seconds to a few minutes. If the tool returns a run that is still RUNNING, read its dataset when it finishes.
- Results are charged per item on the user's Apify account; say so before a large run.
- The skills in `skills/` explain each job (transcripts, comments, profiles, company pages, community research, job listings, trends and more).

# Installing Pom Labs for Cline

Pom Labs is a remote MCP server (streamable HTTP) served by Apify: video transcripts, comments, LinkedIn, Reddit, Google Maps leads, Trustpilot, email verifier and more. Nothing to build or clone.

1. Ask the user for their Apify API token (free account at https://apify.com, Settings > API & Integrations). Do not invent one.
2. Add this to the MCP settings file (`cline_mcp_settings.json`), replacing `<APIFY_TOKEN>`:

```json
{
  "mcpServers": {
    "pomlabs": {
      "type": "streamableHttp",
      "url": "https://mcp.apify.com?tools=pomlabs/social-video-transcript,pomlabs/youtube-transcript-scraper,pomlabs/youtube-comments-scraper,pomlabs/google-maps-business-leads,pomlabs/trustpilot-reviews-scraper,pomlabs/email-verifier,pomlabs/reddit-posts-comments-scraper,pomlabs/linkedin-profile-scraper",
      "headers": { "Authorization": "Bearer <APIFY_TOKEN>" },
      "timeout": 300
    }
  }
}
```

3. Check: the server lists the tools above. Each tool runs a Pom Labs Actor on the user's Apify account and is paid per result (the free Apify credit covers tests).

More tools can be added to the `tools=` list: see https://apify.com/pomlabs

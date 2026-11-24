# Trigger tests

For each skill: three requests that should load it and two similar ones that should not. Run them against the nine descriptions before changing any `description`.

| Skill | Should load | Should not load |
| --- | --- | --- |
| `video-transcripts` | "Summarize this for me: https://youtu.be/jNQXAC9IVRw" · "I need an SRT of this Reel: https://www.instagram.com/reel/Df3Rp0bKNdr/" · "Get the transcripts of these 12 TikToks so I can search them" | "Transcribe this mp3 from my laptop" · "Download this YouTube video as mp4" |
| `video-comments` | "What are viewers asking for under https://www.youtube.com/watch?v=jNQXAC9IVRw ?" · "Pull the 50 most liked comments of this Short" · "Is the reaction to this video positive or negative?" | "Reply to the comments on my own video" · "Get the comments of this Reddit thread" |
| `creator-profile-stats` | "How many followers does @natgeo have on Instagram?" · "Do this creator's Reels usually pass 100k views? instagram.com/nasa" · "Compare these three Instagram accounts by typical Reel views" | "Write a better bio for my Instagram" · "How many subscribers does this YouTube channel have?" |
| `company-pages` | "Enrich these companies with industry and headcount: linkedin.com/company/apify, linkedin.com/company/n8n" · "What has Stripe posted on its LinkedIn page lately?" · "Find companies similar to linkedin.com/company/vercel" | "Find the LinkedIn profile of this person" · "Write a LinkedIn post announcing our launch" |
| `community-research` | "What do people on Reddit dislike about Notion?" · "Get this year's posts mentioning proxies in r/webscraping, with comments" · "What has u/spez posted recently?" | "Draft a Reddit post to launch my app" · "What are people saying in the comments of this YouTube video?" |
| `professional-profiles` | "What is the background of linkedin.com/in/satyanadella?" · "Enrich these candidates with current title and company: linkedin.com/in/williamhgates, linkedin.com/in/jeffweiner08" · "Find people similar to this LinkedIn profile" | "Rewrite my LinkedIn headline" · "How many employees does linkedin.com/company/apify have?" |
| `short-post-search` | "What are people posting on Threads about the new iPhone?" · "Get the replies to https://www.threads.com/@zuck/post/DgHQuRJm8lC" · "How many followers does @zuck have on Threads?" | "Write a Threads post about our launch" · "What do people on Reddit say about the new iPhone?" |
| `marketplace-products` | "Find the best-selling wireless earbuds on AliExpress with prices" · "How much does a 65W GaN charger cost on AliExpress shipped to Brazil?" · "Scrape this AliExpress search page: https://www.aliexpress.com/w/wholesale-robot-vacuum.html" | "Find the cheapest price for this item on Amazon" · "Track my AliExpress order" |
| `video-download` | "Save this TikTok as an MP4: https://www.tiktok.com/@scout2015/video/6718335390845095173" · "I only need the audio of this YouTube video as MP3" · "Download seconds 10 to 25 of this X video" | "What is said in this video? I just need the text" · "Compress this mp4 on my disk to 10 MB" |


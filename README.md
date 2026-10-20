# Scrape LinkedIn Profiles at Scale: B2B Leads Without Sales Navigator

LinkedIn data is the backbone of B2B prospecting — titles, companies, emails, posts. Sales Navigator gives you search; Apify actors give you the export. The top actor (`harvestapi/linkedin-profile-scraper`) has **82k users** and doesn't even need cookies. Here's the pipeline.

> 🤖 **Try it live** — chat with a Poe bot that generates the exact pipeline for your ICP: [Lead List Generator](https://poe.com/leadlistgenerator) · [Job Postings Miner](https://poe.com/jobpostingsminer) · [Social Lead Finder](https://poe.com/socialleadfinder) · [Amazon Product Miner](https://poe.com/amazonproductminer) · [SERP Rank Tracker](https://poe.com/serpranktracker)

## The stack

- **[Apify](https://console.apify.com/sign-up?fpr=dx06p)** — HarvestAPI's no-cookie LinkedIn suite. Free monthly credits.
- **Python** — profiles → scored lead list.

## Step 1 — Token

[Create an Apify account](https://console.apify.com/sign-up?fpr=dx06p) → Settings → Integrations → `export APIFY_TOKEN=...`

## Step 2 — Search → profiles

```bash
curl -X POST "https://api.apify.com/v2/acts/harvestapi~linkedin-profile-search/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "searchQuery": "VP Marketing SaaS",
    "locations": ["United States"],
    "maxItems": 200
  }'
```

Then enrich each profile:

```bash
curl -X POST "https://api.apify.com/v2/acts/harvestapi~linkedin-profile-scraper/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"profileUrls": ["https://www.linkedin.com/in/someone"], "findEmails": true}'
```

## Step 3 — Score for outreach

```python
from apify_client import ApifyClient

ICP_TITLES = ["vp marketing", "head of growth", "cmo", "demand generation"]
client = ApifyClient("APIFY_TOKEN")

run = client.actor("harvestapi/linkedin-profile-search").call(run_input={
    "searchQuery": "growth", "locations": ["United States"], "maxItems": 500,
})

hot = []
for p in client.dataset(run["defaultDatasetId"]).iterate_items():
    title = (p.get("headline") or "").lower()
    if any(t in title for t in ICP_TITLES):
        hot.append({"name": p.get("fullName"), "title": p.get("headline"),
                    "company": p.get("companyName"), "url": p.get("linkedinUrl")})

print(len(hot), "ICP matches")
```

## The plays

- **SDR teams** — search by title+industry → email-enrich → sequenced outreach. Cheaper than Sales Nav + ZoomInfo.
- **Recruiters** — posts scraper (`harvestapi/linkedin-profile-posts`, 42k users) → who's active, whose team is growing.
- **Intel** — competitor employees moving = churn or expansion signal.

## Useful actors

| Actor | Job | Users |
|---|---|---|
| `harvestapi/linkedin-profile-scraper` | Profiles + emails, no cookies | 82.5k |
| `dev_fusion/Linkedin-Profile-Scraper` | Mass profiles + email | 63.7k |
| `harvestapi/linkedin-profile-search` | Search → profiles | 50.8k |
| `harvestapi/linkedin-profile-posts` | Posts/activity data | 42.7k |

## More resources

- [apify-skills](https://github.com/g4dr/apify-skills) — `apify-lead-scraper` skill runs LinkedIn pipelines from a prompt
- [awesome-web-scraping](https://github.com/g4dr/awesome-web-scraping)
- Free credits: [Apify signup](https://console.apify.com/sign-up?fpr=dx06p)

---
name: travel-reviews
description: "Scrape hotel and vacation rental reviews from Expedia, Hotels.com, Vrbo, Agoda and Tripadvisor in one run. Paste your links in any mix: each one is detected and scraped. One dataset with rating (1-5), text, date, reply and source. Runs the Apify Actor carabain-digital/travel-reviews. Use for tasks like: Compare hotel reviews on Expedia, Hotels.com, Tripadvisor; Export Vrbo and Agoda reviews to one CSV or Excel file."
---

# Travel Reviews Scraper

Scrape hotel and vacation rental reviews from Expedia, Hotels.com, Vrbo, Agoda and Tripadvisor in one run. Paste your links in any mix: each one is detected and scraped. One dataset with rating (1-5), text, date, reply and source.

Runs on Apify as the Actor [`carabain-digital/travel-reviews`](https://apify.com/carabain-digital/travel-reviews), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,083 reviews with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~travel-reviews/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~travel-reviews/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"startUrls": [{"url": "https://www.expedia.com/New-York-Hotels-New-York-Marriott-Marquis.h15838.Hotel-Information"}, {"url": "https://www.hotels.com/ho116628/the-lodge-hotel-putney-london-united-kingdom/"}, {"url": "https://www.tripadvisor.com/Hotel_Review-g60763-d93450-Reviews-Hyatt_Grand_Central_New_York-New_York_City_New_York.html"}], "limit": 50}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/travel-reviews").call(run_input={"startUrls": [{"url": "https://www.expedia.com/New-York-Hotels-New-York-Marriott-Marquis.h15838.Hotel-Information"}, {"url": "https://www.hotels.com/ho116628/the-lodge-hotel-putney-london-united-kingdom/"}, {"url": "https://www.tripadvisor.com/Hotel_Review-g60763-d93450-Reviews-Hyatt_Grand_Central_New_York-New_York_City_New_York.html"}], "limit": 50})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/travel-reviews`.

## Input

| Field | Type | Description |
|---|---|---|
| `startUrls` | array | Links to the hotel or vacation rental pages whose reviews you want. Mix sites and add several links per site: the site of each link is detected automatically. Supported: Expedia (expedia.*), Hotels.com, Vrbo, Agoda and Tripadvisor (tripadvisor.*). Links from other sites are skipped with a message in the log. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many reviews for each link (newest first). Ten links with 100 means up to 1,000 reviews. Default: `100`. |
| `cutoff_date` | string | Skip reviews published before this date (YYYY-MM-DD). Useful for incremental runs. |
| `aiAnalysis` | boolean | Adds sentiment (positive, neutral, negative or mixed), a score from -1 to 1, up to 5 topics and a one-line summary to each review with text, using AI. Charged as the AI review analysis add-on per analysed review; reviews without text are not charged. Default: `false`. |

## Examples

**Compare hotel reviews on Expedia, Hotels.com, Tripadvisor**: Fetch the 50 newest reviews of a hotel on each of Expedia, Hotels.com and Tripadvisor into a single dataset with a source column. Replace the example links with the same hotel's pages on each site to compare your rating, reviews and replies across booking sites, then export to CSV, Excel or JSON.

```json
{
  "startUrls": [
    {
      "url": "https://www.expedia.com/New-York-Hotels-New-York-Marriott-Marquis.h15838.Hotel-Information"
    },
    {
      "url": "https://www.hotels.com/ho116628/the-lodge-hotel-putney-london-united-kingdom/"
    },
    {
      "url": "https://www.tripadvisor.com/Hotel_Review-g60763-d93450-Reviews-Hyatt_Grand_Central_New_York-New_York_City_New_York.html"
    }
  ],
  "limit": 50
}
```

**Export Vrbo and Agoda reviews to one CSV or Excel file**: Download the 50 newest reviews per link from a Vrbo vacation rental and an Agoda hotel in one dataset: star rating, review text, date, reviewer, reply and the site each review came from. Replace the example links with your own listings, schedule it weekly to monitor new reviews and export to CSV, Excel or JSON.

```json
{
  "startUrls": [
    {
      "url": "https://www.agoda.com/marina-bay-sands/hotel/singapore-sg.html"
    },
    {
      "url": "https://www.vrbo.com/1309672"
    }
  ],
  "limit": 50
}
```

## Output

One dataset item per result. Main fields: `source`, `published_at`, `rating`, `author`, `title`, `text`, `reply`, `source_url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Review ID prefixed with the site, e.g. expedia:123456; unique across sites and stable across runs |
| `source` | string | Site the review comes from: expedia, hotels_com, vrbo, agoda or tripadvisor |
| `author` | string | Reviewer name as shown on the site |
| `rating` | integer | Rating as 1-5 stars |
| `title` | string | Review title, if the site has one |
| `text` | string | Review text |
| `published_at` | string | Publication date, ISO 8601 UTC |
| `reply` | string | The hotel's, host's or owner's public reply |
| `replied_at` | string | Reply date, ISO 8601 UTC, if published |
| `source_url` | string | The link from your input that this review was scraped from |

## Price

Pay per result: from $2.40 per 1,000 reviews on the Apify Free plan, less on paid plans, plus $0.075 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- Expedia review: $2.88 per 1,000.
- Hotels.com review: $2.88 per 1,000.
- Vrbo review: $3.00 per 1,000.
- Agoda review: $3.36 per 1,000.
- Tripadvisor review: $2.40 per 1,000.
- AI review analysis: $3.00 per 1,000. Sentiment, a score from -1 to 1, topics and a one-line summary for each review with text.

Store page, reviews and full documentation: [Travel Reviews Scraper](https://apify.com/carabain-digital/travel-reviews)

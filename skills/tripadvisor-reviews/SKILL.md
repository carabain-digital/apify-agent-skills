---
name: tripadvisor-reviews
description: "Scrape all reviews of any hotel, restaurant or attraction on Tripadvisor: rating (1-5), text, date, reviewer and reply. Export to JSON, CSV or Excel. Runs the Apify Actor carabain-digital/tripadvisor-reviews. Use for tasks like: Export Tripadvisor reviews to CSV or Excel; Monitor new Tripadvisor reviews of your business."
---

# Tripadvisor Reviews Scraper - Hotels & Restaurants

Scrape all reviews of any hotel, restaurant or attraction on Tripadvisor: rating (1-5), text, date, reviewer and reply. Export to JSON, CSV or Excel.

Runs on Apify as the Actor [`carabain-digital/tripadvisor-reviews`](https://apify.com/carabain-digital/tripadvisor-reviews), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,500 reviews with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~tripadvisor-reviews/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~tripadvisor-reviews/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"url": "https://www.tripadvisor.com/Hotel_Review-g60763-d93450-Reviews-Hyatt_Grand_Central_New_York-New_York_City_New_York.html", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/tripadvisor-reviews").call(run_input={"url": "https://www.tripadvisor.com/Hotel_Review-g60763-d93450-Reviews-Hyatt_Grand_Central_New_York-New_York_City_New_York.html", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/tripadvisor-reviews`.

## Input

| Field | Type | Description |
|---|---|---|
| `startUrls` | array | One or more Tripadvisor review pages, one per hotel, restaurant or attraction. Each link is scraped in turn; a link that fails is skipped and logged. Format: `[{"url": "https://..."}]`. |
| `url` | string | Use this instead of Review page links when you have one link, e.g. in an API call or an older task. The review page of a hotel, restaurant or attraction (..._Review-g<id>-d<id>-Reviews-...), e.g. https://www.tripadvisor.com/Hotel_Review-g60763-d93450-Reviews-Hyatt_Grand_Central_New_York-New_York_City_New_York.html |
| `limit` | integer | Stop after this many reviews for each link, newest first. Leave empty for all. |
| `ratings` | array | Keep only reviews with these star ratings, e.g. 1 and 2 to collect complaints. Leave empty for all ratings. The limit counts the reviews you keep. |
| `cutoff_date` | string | Skip reviews published before this date (YYYY-MM-DD). Useful for incremental runs. |
| `aiAnalysis` | boolean | Adds sentiment (positive, neutral, negative or mixed), a score from -1 to 1, up to 5 topics and a one-line summary to each review with text, using AI. Charged as the AI review analysis add-on per analysed review; reviews without text are not charged. Default: `false`. |

## Examples

**Export Tripadvisor reviews to CSV or Excel**: Download the newest 100 reviews of a hotel, restaurant or attraction on Tripadvisor: star rating, review text, date, reviewer name and the place's reply. Replace the example URL with any place's Tripadvisor page and export the result to CSV, Excel or JSON.

```json
{
  "url": "https://www.tripadvisor.com/Hotel_Review-g60763-d93450-Reviews-Hyatt_Grand_Central_New_York-New_York_City_New_York.html",
  "limit": 100
}
```

**Monitor new Tripadvisor reviews of your business**: Fetch the 20 newest reviews of a hotel, restaurant or attraction on Tripadvisor. Schedule this task daily or weekly to track new ratings and replies, send them to Google Sheets, Slack or your own app, and pay only for the reviews you fetch.

```json
{
  "url": "https://www.tripadvisor.com/Hotel_Review-g60763-d93450-Reviews-Hyatt_Grand_Central_New_York-New_York_City_New_York.html",
  "limit": 20
}
```

## Output

One dataset item per result. Main fields: `published_at`, `rating`, `author`, `title`, `text`, `reply`, `source_url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Tripadvisor review ID, stable across runs |
| `author` | string | Reviewer name as shown on Tripadvisor |
| `rating` | integer | Rating as 1-5 stars |
| `title` | string | Review title, if the site has one |
| `text` | string | Review text |
| `published_at` | string | Publication date, ISO 8601 UTC |
| `reply` | string | The owner's public reply |
| `replied_at` | string | Reply date, ISO 8601 UTC, if published |
| `source_url` | string | The Tripadvisor URL that was scraped |

## Price

Pay per result: $2.00 per 1,000 reviews on the Apify Free plan, less on paid plans, plus $0.025 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- AI review analysis: $3.00 per 1,000. Sentiment, a score from -1 to 1, topics and a one-line summary for each review with text.

Store page, reviews and full documentation: [Tripadvisor Reviews Scraper - Hotels & Restaurants](https://apify.com/carabain-digital/tripadvisor-reviews)

---
name: glassdoor-reviews
description: "Scrape employee reviews of any company on Glassdoor: rating (1-5), pros and cons, date, job title and reply. Export to JSON, CSV or Excel. Runs the Apify Actor carabain-digital/glassdoor-reviews. Use for tasks like: Export Glassdoor reviews to CSV or Excel; Monitor new Glassdoor reviews of your company."
---

# Glassdoor Reviews Scraper

Scrape employee reviews of any company on Glassdoor: rating (1-5), pros and cons, date, job title and reply. Export to JSON, CSV or Excel.

Runs on Apify as the Actor [`carabain-digital/glassdoor-reviews`](https://apify.com/carabain-digital/glassdoor-reviews?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 3,571 reviews with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~glassdoor-reviews/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~glassdoor-reviews/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"url": "https://www.glassdoor.com/Reviews/Glassdoor-Reviews-E100431.htm", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/glassdoor-reviews").call(run_input={"url": "https://www.glassdoor.com/Reviews/Glassdoor-Reviews-E100431.htm", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/glassdoor-reviews`.

## Input

| Field | Type | Description |
|---|---|---|
| `url` | string | The company reviews page (/Reviews/...-E<id>.htm), e.g. https://www.glassdoor.com/Reviews/Glassdoor-Reviews-E100431.htm |
| `limit` | integer | Stop after this many reviews (newest first). Leave empty for all. |
| `cutoff_date` | string | Skip reviews published before this date (YYYY-MM-DD). Useful for incremental runs. |
| `aiAnalysis` | boolean | Adds sentiment (positive, neutral, negative or mixed), a score from -1 to 1, up to 5 topics and a one-line summary to each review with text, using AI. Charged as the AI review analysis add-on per analysed review; reviews without text are not charged. Default: `false`. |

## Examples

**Export Glassdoor reviews to CSV or Excel**: Download the newest 100 reviews of a company on Glassdoor: star rating, review text, date, reviewer name and the company's reply. Replace the example URL with any company's Glassdoor page and export the result to CSV, Excel or JSON.

```json
{
  "url": "https://www.glassdoor.com/Reviews/Glassdoor-Reviews-E100431.htm",
  "limit": 100
}
```

**Monitor new Glassdoor reviews of your company**: Fetch the 20 newest reviews of a company on Glassdoor. Schedule this task daily or weekly to track new ratings and replies, send them to Google Sheets, Slack or your own app, and pay only for the reviews you fetch.

```json
{
  "url": "https://www.glassdoor.com/Reviews/Glassdoor-Reviews-E100431.htm",
  "limit": 20
}
```

## Output

One dataset item per result. Main fields: `published_at`, `rating`, `author`, `title`, `text`, `reply`, `source_url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Glassdoor review ID, stable across runs |
| `author` | string | Reviewer's job title (Glassdoor never shows names), or Anonymous Employee |
| `rating` | integer | Rating as 1-5 stars |
| `title` | string | Review headline |
| `text` | string | Review text: Pros, Cons and Advice to Management, labelled |
| `published_at` | string | Publication date, ISO 8601 UTC |
| `reply` | string | The company's public reply |
| `replied_at` | string | Reply date, ISO 8601 UTC, if published |
| `source_url` | string | The Glassdoor URL that was scraped |

## Price

Pay per result: $1.40 per 1,000 reviews on the Apify Free plan, less on paid plans, plus $0.025 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- AI review analysis: $3.00 per 1,000. Sentiment, a score from -1 to 1, topics and a one-line summary for each review with text.

Store page, reviews and full documentation: [Glassdoor Reviews Scraper](https://apify.com/carabain-digital/glassdoor-reviews?fpr=4ijnom)

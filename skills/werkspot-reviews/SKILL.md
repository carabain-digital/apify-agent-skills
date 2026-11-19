---
name: werkspot-reviews
description: "Scrape all reviews of any tradesperson or company on Werkspot.nl: rating (1-10), job, review text, town, date, first name and the pro's reply. Export to JSON, CSV or Excel. Runs the Apify Actor carabain-digital/werkspot-reviews. Use for tasks like: Export Werkspot reviews to CSV or Excel; Monitor new Werkspot reviews of your business."
---

# Werkspot Reviews Scraper

Scrape all reviews of any tradesperson or company on Werkspot.nl: rating (1-10), job, review text, town, date, first name and the pro's reply. Export to JSON, CSV or Excel.

Runs on Apify as the Actor [`carabain-digital/werkspot-reviews`](https://apify.com/carabain-digital/werkspot-reviews), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,500 reviews with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~werkspot-reviews/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~werkspot-reviews/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"url": "https://www.werkspot.nl/profiel/multi-loodgieter", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/werkspot-reviews").call(run_input={"url": "https://www.werkspot.nl/profiel/multi-loodgieter", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/werkspot-reviews`.

## Input

| Field | Type | Description |
|---|---|---|
| `url` | string | The company's Werkspot profile, e.g. https://www.werkspot.nl/profiel/multi-loodgieter (its /reviews page works too) |
| `limit` | integer | Stop after this many reviews (newest first). Leave empty for all. |
| `cutoff_date` | string | Skip reviews published before this date (YYYY-MM-DD). Useful for incremental runs. |
| `aiAnalysis` | boolean | Adds sentiment (positive, neutral, negative or mixed), a score from -1 to 1, up to 5 topics and a one-line summary to each review with text, using AI. Charged as the AI review analysis add-on per analysed review; reviews without text are not charged. Default: `false`. |

## Examples

**Export Werkspot reviews to CSV or Excel**: Download the newest 100 reviews of a tradesperson or company on Werkspot.nl: rating, job, review text, town, date and the pro's reply. Replace the example URL with any Werkspot profile and export the result to CSV, Excel or JSON.

```json
{
  "url": "https://www.werkspot.nl/profiel/multi-loodgieter",
  "limit": 100
}
```

**Monitor new Werkspot reviews of your business**: Fetch the 20 newest reviews of a tradesperson or company on Werkspot.nl. Schedule this task daily or weekly to track new ratings of your business or your competitors, send them to Google Sheets, Slack or your own app, and pay only for the reviews you fetch.

```json
{
  "url": "https://www.werkspot.nl/profiel/multi-loodgieter",
  "limit": 20
}
```

## Output

One dataset item per result. Main fields: `published_at`, `rating_exact`, `title`, `text`, `author`, `location`, `reply`, `source_url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Werkspot review ID, stable across runs |
| `author` | string | Reviewer's first name, or Anonymous |
| `rating` | integer | Rating on a 1-5 scale |
| `title` | string | The job that was reviewed, e.g. 'Radiatorkranen: 2 kranen; Vervangen' |
| `text` | string | Review text (can be empty: many reviews are rating only) |
| `published_at` | string | Publication date and time, ISO 8601 (UTC) |
| `reply` | string | The tradesperson's reply, if any |
| `replied_at` | string | Date and time of the reply, ISO 8601 (UTC) |
| `rating_exact` | number | Rating as given, 1-10 |
| `location` | string | Town where the job was done |

## Price

Pay per result: $2.00 per 1,000 reviews on the Apify Free plan, less on paid plans, plus $0.025 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- AI review analysis: $3.00 per 1,000. Sentiment, a score from -1 to 1, topics and a one-line summary for each review with text.

Store page, reviews and full documentation: [Werkspot Reviews Scraper](https://apify.com/carabain-digital/werkspot-reviews)

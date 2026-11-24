---
name: golocal-reviews
description: "Scrape all reviews of any German business on golocal.de, including reviews from MeinungsMeister, Yelp and other partner sites: rating, criteria ratings, text, author, date and source. Export to JSON, CSV or Excel. Runs the Apify Actor carabain-digital/golocal-reviews. Use for tasks like: Export golocal reviews to CSV or Excel; Monitor new golocal reviews of your business."
---

# golocal Reviews Scraper

Scrape all reviews of any German business on golocal.de, including reviews from MeinungsMeister, Yelp and other partner sites: rating, criteria ratings, text, author, date and source. Export to JSON, CSV or Excel.

Runs on Apify as the Actor [`carabain-digital/golocal-reviews`](https://apify.com/carabain-digital/golocal-reviews?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,500 reviews with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~golocal-reviews/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~golocal-reviews/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"url": "https://www.golocal.de/paderborn/autohaeuser/krenz-autohaus-werkstatt-10geo3/", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/golocal-reviews").call(run_input={"url": "https://www.golocal.de/paderborn/autohaeuser/krenz-autohaus-werkstatt-10geo3/", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/golocal-reviews`.

## Input

| Field | Type | Description |
|---|---|---|
| `url` | string | The business page on golocal.de, e.g. https://www.golocal.de/paderborn/autohaeuser/krenz-autohaus-werkstatt-10geo3/ (its /bewertungen/ page works too) |
| `limit` | integer | Stop after this many reviews (newest first). Leave empty for all. |
| `cutoff_date` | string | Skip reviews published before this date (YYYY-MM-DD). Useful for incremental runs. |
| `aiAnalysis` | boolean | Adds sentiment (positive, neutral, negative or mixed), a score from -1 to 1, up to 5 topics and a one-line summary to each review with text, using AI. Charged as the AI review analysis add-on per analysed review; reviews without text are not charged. Default: `false`. |

## Examples

**Export golocal reviews to CSV or Excel**: Download the newest 100 reviews of a German business on golocal.de, including reviews from MeinungsMeister, Yelp and other partner sites: rating, criteria ratings, text, author, date and source. Replace the example URL with any business on golocal and export to CSV, Excel or JSON.

```json
{
  "url": "https://www.golocal.de/paderborn/autohaeuser/krenz-autohaus-werkstatt-10geo3/",
  "limit": 100
}
```

**Monitor new golocal reviews of your business**: Fetch the 20 newest reviews of a German business on golocal.de. Schedule this task daily or weekly to track new ratings of your business or your competitors, send them to Google Sheets, Slack or your own app, and pay only for the reviews you fetch.

```json
{
  "url": "https://www.golocal.de/paderborn/autohaeuser/krenz-autohaus-werkstatt-10geo3/",
  "limit": 20
}
```

## Output

One dataset item per result. Main fields: `published_at`, `rating_exact`, `text`, `author`, `source`, `source_url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | golocal review ID, stable across runs |
| `author` | string | Reviewer name, or Anonymous |
| `rating` | integer | Rating on a 1-5 scale |
| `title` | string | Always empty: golocal reviews have no title |
| `text` | string | Review text (reviews imported from partner sites can be an excerpt) |
| `published_at` | string | Publication date and time, ISO 8601 (UTC) |
| `reply` | string | Always null: golocal shows no replies from the business |
| `replied_at` | string | Always null |
| `rating_exact` | number | Rating as shown, 1-5 with decimals |
| `rating_criteria` | object | Ratings per criterion (e.g. Service, Angebot, Preis/Leistung), when given |
| `source` | string | Where the review was written: golocal or a partner site (meinungsmeister, yelp, speisekarte, ...) |

## Price

Pay per result: $2.00 per 1,000 reviews on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- AI review analysis: $3.00 per 1,000. Sentiment, a score from -1 to 1, topics and a one-line summary for each review with text.

Store page, reviews and full documentation: [golocal Reviews Scraper](https://apify.com/carabain-digital/golocal-reviews?fpr=4ijnom)

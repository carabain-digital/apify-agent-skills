---
name: kieskeurig-reviews
description: "Scrape all reviews of any product or webshop on Kieskeurig.nl: rating (1-10 and 1-5), title, text, pros, cons, author and date. Export to JSON, CSV or Excel. Runs the Apify Actor carabain-digital/kieskeurig-reviews. Use for tasks like: Export Kieskeurig product reviews to CSV or Excel; Monitor new Kieskeurig reviews of your webshop."
---

# Kieskeurig Reviews Scraper

Scrape all reviews of any product or webshop on Kieskeurig.nl: rating (1-10 and 1-5), title, text, pros, cons, author and date. Export to JSON, CSV or Excel.

Runs on Apify as the Actor [`carabain-digital/kieskeurig-reviews`](https://apify.com/carabain-digital/kieskeurig-reviews), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,500 reviews with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~kieskeurig-reviews/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~kieskeurig-reviews/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"url": "https://www.kieskeurig.nl/bijverwarming/product/51284338-philips-compacte-keramische-verwarming", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/kieskeurig-reviews").call(run_input={"url": "https://www.kieskeurig.nl/bijverwarming/product/51284338-philips-compacte-keramische-verwarming", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/kieskeurig-reviews`.

## Input

| Field | Type | Description |
|---|---|---|
| `url` | string | A product page (https://www.kieskeurig.nl/<category>/product/<id>-<name>) or a shop page (https://www.kieskeurig.nl/winkels/<shop>). The /reviews page of either works too. |
| `limit` | integer | Stop after this many reviews (newest first). Leave empty for all. |
| `cutoff_date` | string | Skip reviews published before this date (YYYY-MM-DD). Useful for incremental runs. |
| `aiAnalysis` | boolean | Adds sentiment (positive, neutral, negative or mixed), a score from -1 to 1, up to 5 topics and a one-line summary to each review with text, using AI. Charged as the AI review analysis add-on per analysed review; reviews without text are not charged. Default: `false`. |

## Examples

**Export Kieskeurig product reviews to CSV or Excel**: Download the newest 100 reviews of a product on Kieskeurig.nl: rating, title, text, pros, cons, author and date. Replace the example URL with any product page on Kieskeurig and export the result to CSV, Excel or JSON.

```json
{
  "url": "https://www.kieskeurig.nl/bijverwarming/product/51284338-philips-compacte-keramische-verwarming",
  "limit": 100
}
```

**Monitor new Kieskeurig reviews of your webshop**: Fetch the 20 newest reviews of a webshop on Kieskeurig.nl. Schedule this task daily or weekly to track new ratings of your shop or your competitors, send them to Google Sheets, Slack or your own app, and pay only for the reviews you fetch.

```json
{
  "url": "https://www.kieskeurig.nl/winkels/bol./reviews",
  "limit": 20
}
```

## Output

One dataset item per result. Main fields: `published_at`, `rating_exact`, `title`, `text`, `pros`, `cons`, `author`, `source_url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Review ID derived from date, author and text; stable across runs |
| `author` | string | Reviewer name as Kieskeurig shows it (often masked) |
| `rating` | integer | Rating on a 1-5 scale |
| `title` | string | Review title (product reviews) |
| `text` | string | Review text |
| `published_at` | string | Publication date, ISO 8601 (UTC) |
| `reply` | string | Always null: Kieskeurig shows no replies |
| `replied_at` | string | Always null |
| `rating_exact` | number | Rating as given, 1-10 |
| `pros` | array | Pros the reviewer listed |
| `cons` | array | Cons the reviewer listed |

## Price

Pay per result: $2.00 per 1,000 reviews on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- AI review analysis: $3.00 per 1,000. Sentiment, a score from -1 to 1, topics and a one-line summary for each review with text.

Store page, reviews and full documentation: [Kieskeurig Reviews Scraper](https://apify.com/carabain-digital/kieskeurig-reviews)

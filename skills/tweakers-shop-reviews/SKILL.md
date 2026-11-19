---
name: tweakers-shop-reviews
description: "Scrape all reviews of any webshop on Tweakers.net Pricewatch: overall and per-aspect ratings (shipping, customer service), full text per aspect, author, date and the shop's reply. Export to JSON, CSV or Excel. Runs the Apify Actor carabain-digital/tweakers-shop-reviews. Use for tasks like: Export Tweakers shop reviews to CSV or Excel; Monitor new Tweakers reviews of your webshop."
---

# Tweakers Shop Reviews Scraper

Scrape all reviews of any webshop on Tweakers.net Pricewatch: overall and per-aspect ratings (shipping, customer service), full text per aspect, author, date and the shop's reply. Export to JSON, CSV or Excel.

Runs on Apify as the Actor [`carabain-digital/tweakers-shop-reviews`](https://apify.com/carabain-digital/tweakers-shop-reviews), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,500 reviews with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~tweakers-shop-reviews/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~tweakers-shop-reviews/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"url": "https://tweakers.net/shopreview/5382/bol/overzicht/", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/tweakers-shop-reviews").call(run_input={"url": "https://tweakers.net/shopreview/5382/bol/overzicht/", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/tweakers-shop-reviews`.

## Input

| Field | Type | Description |
|---|---|---|
| `url` | string | The webshop's review page on Tweakers, e.g. https://tweakers.net/shopreview/5382/bol/overzicht/ (the /shopreviews/ page works too) |
| `limit` | integer | Stop after this many reviews (newest first). Leave empty for all. |
| `cutoff_date` | string | Skip reviews published before this date (YYYY-MM-DD). Useful for incremental runs. |
| `aiAnalysis` | boolean | Adds sentiment (positive, neutral, negative or mixed), a score from -1 to 1, up to 5 topics and a one-line summary to each review with text, using AI. Charged as the AI review analysis add-on per analysed review; reviews without text are not charged. Default: `false`. |

## Examples

**Export Tweakers shop reviews to CSV or Excel**: Download the newest 100 reviews of a webshop on Tweakers.net: overall, shipping and customer service ratings, full review text, author, date and the shop's reply. Replace the example URL with any shop's Tweakers review page and export to CSV, Excel or JSON.

```json
{
  "url": "https://tweakers.net/shopreview/5382/bol/overzicht/",
  "limit": 100
}
```

**Monitor new Tweakers reviews of your webshop**: Fetch the 20 newest reviews of a webshop on Tweakers.net. Schedule this task daily or weekly to track new ratings and replies for your shop or your competitors, send them to Google Sheets, Slack or your own app, and pay only for the reviews you fetch.

```json
{
  "url": "https://tweakers.net/shopreview/5382/bol/overzicht/",
  "limit": 20
}
```

## Output

One dataset item per result. Main fields: `published_at`, `rating`, `text`, `author`, `reply`, `source_url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Tweakers review ID, stable across runs |
| `author` | string | Tweakers username of the reviewer |
| `rating` | integer | Overall rating (Algemeen), 1-5 |
| `title` | string | Always empty: shop reviews have no title |
| `text` | string | Full review text, all sections |
| `published_at` | string | Publication date and time, ISO 8601 (UTC) |
| `reply` | string | The webshop's reply, if any |
| `replied_at` | string | Date and time of the reply, ISO 8601 (UTC) |
| `reply_author` | string | Tweakers account the shop replied with |
| `rating_exact` | number | Overall rating as given, 1-5 |
| `rating_aspects` | object | Star rating per aspect: Algemeen, Verzending, Klantenservice, ... |
| `sections` | object | Review text per aspect: Algemeen, Verzending, Klantenservice, ... |
| `motivation` | string | Why the review was written, e.g. 'Uit eigen beweging' (own initiative) |

## Price

Pay per result: $2.00 per 1,000 reviews on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- AI review analysis: $3.00 per 1,000. Sentiment, a score from -1 to 1, topics and a one-line summary for each review with text.

Store page, reviews and full documentation: [Tweakers Shop Reviews Scraper](https://apify.com/carabain-digital/tweakers-shop-reviews)

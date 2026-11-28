---
name: amazon-seller-reviews
description: "Scrape all feedback of any seller on Amazon: rating (1-5), text, date, reviewer and seller reply. Export to JSON, CSV or Excel. Runs the Apify Actor carabain-digital/amazon-seller-reviews. Use for tasks like: Export Amazon Seller reviews to CSV or Excel; Monitor new Amazon Seller reviews of your seller."
---

# Amazon Seller Reviews Scraper - Seller Feedback

Scrape all feedback of any seller on Amazon: rating (1-5), text, date, reviewer and seller reply. Export to JSON, CSV or Excel.

Runs on Apify as the Actor [`carabain-digital/amazon-seller-reviews`](https://apify.com/carabain-digital/amazon-seller-reviews?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,500 reviews with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~amazon-seller-reviews/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~amazon-seller-reviews/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"url": "https://www.amazon.com/sp?seller=A294P4X9EWVXLJ", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/amazon-seller-reviews").call(run_input={"url": "https://www.amazon.com/sp?seller=A294P4X9EWVXLJ", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/amazon-seller-reviews`.

## Input

| Field | Type | Description |
|---|---|---|
| `startUrls` | array | One or more Amazon seller pages (https://www.amazon.com/sp?seller=...), one per seller. Each link is scraped in turn; a link that fails is skipped and logged. Format: `[{"url": "https://..."}]`. |
| `url` | string | Use this instead of Review page links when you have one link, e.g. in an API call or an older task. The seller page, e.g. https://www.amazon.com/sp?seller=A294P4X9EWVXLJ. Other Amazon domains work too (amazon.de, amazon.co.uk, ...). |
| `seller` | string | Use this with Domain instead of a link, e.g. A294P4X9EWVXLJ. It is the value after seller= in the seller page link. |
| `domain` | string | Amazon site for Seller ID, e.g. www.amazon.de. Default: www.amazon.com. |
| `marketplace_id` | string | Amazon marketplace ID, e.g. ATVPDKIKX0DER for amazon.com. Found automatically for the main Amazon sites; only set it for others. |
| `limit` | integer | Stop after this many reviews for each link, newest first. Leave empty for all. |
| `cutoff_date` | string | Skip reviews published before this date (YYYY-MM-DD). Useful for incremental runs. |
| `asin` | string | Keep only feedback about this product, e.g. B07XYZ1234. |
| `time_period` | string | Which feedback period Amazon returns. Default: Lifetime, the only value tested. Default: `"Lifetime"`. |
| `from_rating` | integer | Ask Amazon for feedback from this rating up, 1-5, e.g. 1 with Highest star rating 2 to collect complaints. Leave empty for all. |
| `to_rating` | integer | Ask Amazon for feedback up to this rating, 1-5. Leave empty for all. |
| `include_partially_suppressed` | boolean | Also return feedback Amazon partly hides, e.g. text removed after a dispute. On by default. Default: `true`. |
| `max_pages` | integer | Stop after this many pages of 5 reviews. Default: 100. Default: `100`. |
| `start_page` | integer | First page to fetch, counting from 0. Default: 0. Default: `0`. |
| `aiAnalysis` | boolean | Adds sentiment (positive, neutral, negative or mixed), a score from -1 to 1, up to 5 topics and a one-line summary to each review with text, using AI. Charged as the AI review analysis add-on per analysed review; reviews without text are not charged. Default: `false`. |

## Examples

**Export Amazon Seller reviews to CSV or Excel**: Download the newest 100 reviews of a seller on Amazon Seller: star rating, review text, date, reviewer name and the seller's reply. Replace the example URL with any seller's Amazon Seller page and export the result to CSV, Excel or JSON.

```json
{
  "url": "https://www.amazon.com/sp?seller=A294P4X9EWVXLJ",
  "limit": 100
}
```

**Monitor new Amazon Seller reviews of your seller**: Fetch the 20 newest reviews of a seller on Amazon Seller. Schedule this task daily or weekly to track new ratings and replies, send them to Google Sheets, Slack or your own app, and pay only for the reviews you fetch.

```json
{
  "url": "https://www.amazon.com/sp?seller=A294P4X9EWVXLJ",
  "limit": 20
}
```

## Output

One dataset item per result. Main fields: `published_at`, `rating`, `author`, `title`, `text`, `reply`, `source_url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Review ID, a hash of seller, reviewer, date and text; stable across runs |
| `author` | string | Reviewer name as shown on Amazon |
| `rating` | integer | Rating as 1-5 stars |
| `title` | string | Always null: Amazon seller feedback has no title |
| `text` | string | Review text |
| `published_at` | string | Publication date, ISO 8601 UTC |
| `reply` | string | The seller's public reply |
| `replied_at` | string | Reply date, ISO 8601 UTC, if published |
| `source_url` | string | The Amazon seller page that was scraped |

## Price

Pay per result: $2.00 per 1,000 reviews on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- AI review analysis: $3.00 per 1,000. Sentiment, a score from -1 to 1, topics and a one-line summary for each review with text.

Store page, reviews and full documentation: [Amazon Seller Reviews Scraper - Seller Feedback](https://apify.com/carabain-digital/amazon-seller-reviews?fpr=4ijnom)

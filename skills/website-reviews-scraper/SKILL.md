---
name: website-reviews-scraper
description: "Scrape the reviews shown on any webshop or business website: Judge.me, Yotpo, Reviews.io, Okendo, Stamped, Feefo, eKomi, Trusted Shops, ProvenExpert, Feedback Company, Bazaarvoice, PowerReviews and Loox. Paste a URL, get every review. Runs the Apify Actor carabain-digital/website-reviews-scraper. Use for tasks like: Export a webshop's reviews to CSV or Excel; Monitor new reviews on your webshop."
---

# Website Reviews Scraper

Scrape the reviews shown on any webshop or business website: Judge.me, Yotpo, Reviews.io, Okendo, Stamped, Feefo, eKomi, Trusted Shops, ProvenExpert, Feedback Company, Bazaarvoice, PowerReviews and Loox. Paste a URL, get every review.

Runs on Apify as the Actor [`carabain-digital/website-reviews-scraper`](https://apify.com/carabain-digital/website-reviews-scraper?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 5,000 reviews with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~website-reviews-scraper/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~website-reviews-scraper/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"url": "https://www.brooklinen.com", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/website-reviews-scraper").call(run_input={"url": "https://www.brooklinen.com", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/website-reviews-scraper`.

## Input

| Field | Type | Description |
|---|---|---|
| `url` | string | The homepage (or any page) of a webshop or business that shows reviews from Judge.me, Yotpo, Reviews.io, Okendo, Stamped, Feefo, eKomi, Trusted Shops, ProvenExpert, Feedback Company, Bazaarvoice, PowerReviews or Loox. For Bazaarvoice and PowerReviews use a product page. A link to the platform's own review page works too, e.g. https://www.ekomi.de/bewertungen-bettenrid.html |
| `limit` | integer | Stop after this many reviews (newest first). Leave empty for all. |
| `cutoff_date` | string | Skip reviews published before this date (YYYY-MM-DD). Useful for incremental runs. |
| `platforms` | array | Leave empty to scrape every review platform found on the site. |
| `aiAnalysis` | boolean | Adds sentiment (positive, neutral, negative or mixed), a score from -1 to 1, up to 5 topics and a one-line summary to each review with text, using AI. Charged as the AI review analysis add-on per analysed review; reviews without text are not charged. Default: `false`. |

## Examples

**Export a webshop's reviews to CSV or Excel**: Download the newest 100 reviews shown on a webshop: star rating, review text, date, reviewer name, product and the shop's reply. Works with Judge.me, Yotpo, Reviews.io, Okendo, Stamped, Feefo, eKomi, Trusted Shops, ProvenExpert and Feedback Company. Replace the example URL with any shop's homepage.

```json
{
  "url": "https://www.brooklinen.com",
  "limit": 100
}
```

**Monitor new reviews on your webshop**: Fetch the 20 newest reviews shown on a webshop or business website. Schedule this task daily or weekly to track new ratings and replies, send them to Google Sheets, Slack or your own app, and pay only for the reviews you fetch.

```json
{
  "url": "https://www.brooklinen.com",
  "limit": 20
}
```

## Output

One dataset item per result. Main fields: `published_at`, `rating`, `platform`, `review_type`, `product`, `author`, `title`, `text`, `reply`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Platform-prefixed review ID (e.g. yotpo:914498538), stable across runs |
| `author` | string | Reviewer name as shown by the platform |
| `rating` | integer | Rating as 1-5 stars |
| `title` | string | Review title, if the site has one |
| `text` | string | Review text |
| `published_at` | string | Publication date, ISO 8601 UTC |
| `reply` | string | The shop's public reply |
| `replied_at` | string | Reply date, ISO 8601 UTC, if published |
| `source_url` | string | The website URL that was scraped |
| `platform` | string | Review platform: judgeme, yotpo, reviewsio, okendo, stamped, feefo, ekomi, trustedshops, provenexpert, feedbackcompany, bazaarvoice, powerreviews or loox |
| `review_type` | string | store = review of the shop or company, product = review of one product |
| `product` | string | Product name, for product reviews |
| `product_url` | string | Product link, for product reviews |

## Price

Pay per result: $1.00 per 1,000 reviews on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- AI review analysis: $3.00 per 1,000. Sentiment, a score from -1 to 1, topics and a one-line summary for each review with text.

Store page, reviews and full documentation: [Website Reviews Scraper](https://apify.com/carabain-digital/website-reviews-scraper?fpr=4ijnom)

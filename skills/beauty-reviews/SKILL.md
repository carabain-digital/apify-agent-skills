---
name: beauty-reviews
description: "Scrape client reviews of salons, barbers, spas and fitness studios from Booksy, Fresha, StyleSeat, Vagaro, Mindbody and ClassPass in one run. Paste your links in any mix: each one is detected and scraped. One dataset with rating (1-5), text, date, reply and source. Runs the Apify Actor carabain-digital/beauty-reviews. Use for tasks like: Compare salon reviews: Booksy, Fresha, StyleSeat, Vagaro; Export Mindbody and ClassPass studio reviews to CSV or Excel."
---

# Beauty & Wellness Reviews Scraper

Scrape client reviews of salons, barbers, spas and fitness studios from Booksy, Fresha, StyleSeat, Vagaro, Mindbody and ClassPass in one run. Paste your links in any mix: each one is detected and scraped. One dataset with rating (1-5), text, date, reply and source.

Runs on Apify as the Actor [`carabain-digital/beauty-reviews`](https://apify.com/carabain-digital/beauty-reviews?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,976 reviews with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~beauty-reviews/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~beauty-reviews/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"startUrls": [{"url": "https://booksy.com/en-us/110442_gramercy-best-barbers_barber-shop_30067_new-york-city"}, {"url": "https://www.fresha.com/a/venue-salon-lakeland-5035-south-lakeland-drive-brznh7lm"}, {"url": "https://www.styleseat.com/m/v/leahlang"}, {"url": "https://www.vagaro.com/styledsalonspa"}], "limit": 50}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/beauty-reviews").call(run_input={"startUrls": [{"url": "https://booksy.com/en-us/110442_gramercy-best-barbers_barber-shop_30067_new-york-city"}, {"url": "https://www.fresha.com/a/venue-salon-lakeland-5035-south-lakeland-drive-brznh7lm"}, {"url": "https://www.styleseat.com/m/v/leahlang"}, {"url": "https://www.vagaro.com/styledsalonspa"}], "limit": 50})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/beauty-reviews`.

## Input

| Field | Type | Description |
|---|---|---|
| `startUrls` | array | Links to the salon, barber, spa or studio pages whose reviews you want. Mix sites and add several links per site: the site of each link is detected automatically. Supported: Booksy, Fresha, StyleSeat, Vagaro, Mindbody and ClassPass. Links from other sites are skipped with a message in the log. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many reviews for each link (newest first). Ten links with 100 means up to 1,000 reviews. Default: `100`. |
| `cutoff_date` | string | Skip reviews published before this date (YYYY-MM-DD). Useful for incremental runs. |
| `aiAnalysis` | boolean | Adds sentiment (positive, neutral, negative or mixed), a score from -1 to 1, up to 5 topics and a one-line summary to each review with text, using AI. Charged as the AI review analysis add-on per analysed review; reviews without text are not charged. Default: `false`. |

## Examples

**Compare salon reviews: Booksy, Fresha, StyleSeat, Vagaro**: Fetch the 50 newest reviews per link from Booksy, Fresha, StyleSeat and Vagaro into a single dataset with a source column. Replace the example links with your salon, spa or studio's pages on each site to compare ratings, reviews and replies across platforms, then export to CSV, Excel or JSON.

```json
{
  "startUrls": [
    {
      "url": "https://booksy.com/en-us/110442_gramercy-best-barbers_barber-shop_30067_new-york-city"
    },
    {
      "url": "https://www.fresha.com/a/venue-salon-lakeland-5035-south-lakeland-drive-brznh7lm"
    },
    {
      "url": "https://www.styleseat.com/m/v/leahlang"
    },
    {
      "url": "https://www.vagaro.com/styledsalonspa"
    }
  ],
  "limit": 50
}
```

**Export Mindbody and ClassPass studio reviews to CSV or Excel**: Download the 50 newest reviews per link from Mindbody and ClassPass in one dataset: star rating, review text, date, reviewer and the site each review came from. Replace the example links with your own, schedule it weekly to monitor new reviews and export to CSV, Excel or JSON.

```json
{
  "startUrls": [
    {
      "url": "https://www.mindbodyonline.com/explore/locations/mighty-pilates-santa-monica"
    },
    {
      "url": "https://classpass.com/studios/barrys-brooklyn-heights"
    }
  ],
  "limit": 50
}
```

## Output

One dataset item per result. Main fields: `source`, `published_at`, `rating`, `author`, `title`, `text`, `reply`, `source_url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Review ID prefixed with the site, e.g. booksy:abc123; unique across sites and stable across runs |
| `source` | string | Site the review comes from: booksy, fresha, styleseat, vagaro, mindbody or classpass |
| `author` | string | Reviewer name as shown on the site |
| `rating` | integer | Rating as 1-5 stars |
| `title` | string | Review title, if the site has one |
| `text` | string | Review text |
| `published_at` | string | Publication date, ISO 8601 UTC |
| `reply` | string | The business's public reply |
| `replied_at` | string | Reply date, ISO 8601 UTC, if published |
| `source_url` | string | The link from your input that this review was scraped from |

## Price

Pay per result: from $1.68 per 1,000 reviews on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Booksy review: $1.68 per 1,000.
- Fresha review: $2.40 per 1,000.
- StyleSeat review: $2.40 per 1,000.
- Vagaro review: $2.40 per 1,000.
- Mindbody review: $2.40 per 1,000.
- ClassPass review: $2.40 per 1,000.
- AI review analysis: $3.00 per 1,000. Sentiment, a score from -1 to 1, topics and a one-line summary for each review with text.

Store page, reviews and full documentation: [Beauty & Wellness Reviews Scraper](https://apify.com/carabain-digital/beauty-reviews?fpr=4ijnom)

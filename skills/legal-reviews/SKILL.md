---
name: legal-reviews
description: "Scrape lawyer and law firm reviews from Avvo, FindLaw, Lawyers.com, Martindale and ReviewSolicitors in one run. Paste your links in any mix: each one is detected and scraped. One dataset with rating (1-5), text, date, reply and source. Runs the Apify Actor carabain-digital/legal-reviews. Use for tasks like: Compare lawyer reviews: Avvo, FindLaw, Martindale, Lawyers.com; Export law firm reviews: Martindale and ReviewSolicitors."
---

# Legal Reviews Scraper

Scrape lawyer and law firm reviews from Avvo, FindLaw, Lawyers.com, Martindale and ReviewSolicitors in one run. Paste your links in any mix: each one is detected and scraped. One dataset with rating (1-5), text, date, reply and source.

Runs on Apify as the Actor [`carabain-digital/legal-reviews`](https://apify.com/carabain-digital/legal-reviews), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 4,901 reviews with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~legal-reviews/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~legal-reviews/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"startUrls": [{"url": "https://www.avvo.com/attorneys/89101-nv-daniel-simon-4283044.html"}, {"url": "https://lawyers.findlaw.com/new-york/brooklyn/shua-gelbstein-NTM5Mjc3OV8x/"}, {"url": "https://www.lawyers.com/arlington/texas/bailey-galyen-attorneys-at-law-300013843-f/"}, {"url": "https://www.martindale.com/organization/bailey-galyen-attorneys-at-law-1297222/arlington-texas-300013843-f/"}], "limit": 50}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/legal-reviews").call(run_input={"startUrls": [{"url": "https://www.avvo.com/attorneys/89101-nv-daniel-simon-4283044.html"}, {"url": "https://lawyers.findlaw.com/new-york/brooklyn/shua-gelbstein-NTM5Mjc3OV8x/"}, {"url": "https://www.lawyers.com/arlington/texas/bailey-galyen-attorneys-at-law-300013843-f/"}, {"url": "https://www.martindale.com/organization/bailey-galyen-attorneys-at-law-1297222/arlington-texas-300013843-f/"}], "limit": 50})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/legal-reviews`.

## Input

| Field | Type | Description |
|---|---|---|
| `startUrls` | array | Links to the lawyer or law firm pages whose reviews you want. Mix sites and add several links per site: the site of each link is detected automatically. Supported: Avvo, FindLaw (lawyers.findlaw.com), Lawyers.com, Martindale and ReviewSolicitors (reviewsolicitors.co.uk). Links from other sites are skipped with a message in the log. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many reviews for each link (newest first). Ten links with 100 means up to 1,000 reviews. Default: `100`. |
| `cutoff_date` | string | Skip reviews published before this date (YYYY-MM-DD). Useful for incremental runs. |
| `aiAnalysis` | boolean | Adds sentiment (positive, neutral, negative or mixed), a score from -1 to 1, up to 5 topics and a one-line summary to each review with text, using AI. Charged as the AI review analysis add-on per analysed review; reviews without text are not charged. Default: `false`. |

## Examples

**Compare lawyer reviews: Avvo, FindLaw, Martindale, Lawyers.com**: Fetch the 50 newest reviews per link from Avvo, FindLaw, Martindale and Lawyers.com into a single dataset with a source column. Replace the example links with your lawyer's or law firm's pages on each site to compare ratings, reviews and replies across platforms, then export to CSV, Excel or JSON.

```json
{
  "startUrls": [
    {
      "url": "https://www.avvo.com/attorneys/89101-nv-daniel-simon-4283044.html"
    },
    {
      "url": "https://lawyers.findlaw.com/new-york/brooklyn/shua-gelbstein-NTM5Mjc3OV8x/"
    },
    {
      "url": "https://www.lawyers.com/arlington/texas/bailey-galyen-attorneys-at-law-300013843-f/"
    },
    {
      "url": "https://www.martindale.com/organization/bailey-galyen-attorneys-at-law-1297222/arlington-texas-300013843-f/"
    }
  ],
  "limit": 50
}
```

**Export law firm reviews: Martindale and ReviewSolicitors**: Download the 50 newest client reviews per link from Martindale, Lawyers.com and ReviewSolicitors (UK) in one dataset: star rating, review text, date, reviewer and the site each review came from. Replace the example links with your own firms, schedule it weekly to monitor new reviews and export to CSV, Excel or JSON.

```json
{
  "startUrls": [
    {
      "url": "https://www.martindale.com/organization/bailey-galyen-attorneys-at-law-1297222/arlington-texas-300013843-f/"
    },
    {
      "url": "https://www.lawyers.com/arlington/texas/bailey-galyen-attorneys-at-law-300013843-f/"
    },
    {
      "url": "https://www.reviewsolicitors.co.uk/buckinghamshire/milton-keynes/heald-solicitors-llp"
    }
  ],
  "limit": 50
}
```

## Output

One dataset item per result. Main fields: `source`, `published_at`, `rating`, `author`, `title`, `text`, `reply`, `source_url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Review ID prefixed with the site, e.g. avvo:abc123; unique across sites and stable across runs |
| `source` | string | Site the review comes from: avvo, findlaw, lawyers_com, martindale or reviewsolicitors |
| `author` | string | Reviewer name as shown on the site |
| `rating` | integer | Rating as 1-5 stars |
| `title` | string | Review title, if the site has one |
| `text` | string | Review text |
| `published_at` | string | Publication date, ISO 8601 UTC |
| `reply` | string | The lawyer's or law firm's public reply |
| `replied_at` | string | Reply date, ISO 8601 UTC, if published |
| `source_url` | string | The link from your input that this review was scraped from |

## Price

Pay per result: from $1.02 per 1,000 reviews on the Apify Free plan, less on paid plans, plus $0.025 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- Avvo review: $1.68 per 1,000.
- FindLaw review: $1.02 per 1,000.
- Lawyers.com review: $3.60 per 1,000.
- Martindale review: $3.60 per 1,000.
- ReviewSolicitors review: $2.40 per 1,000.
- AI review analysis: $3.00 per 1,000. Sentiment, a score from -1 to 1, topics and a one-line summary for each review with text.

Store page, reviews and full documentation: [Legal Reviews Scraper](https://apify.com/carabain-digital/legal-reviews)

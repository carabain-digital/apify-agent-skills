---
name: home-services-reviews
description: "Scrape customer reviews of contractors and home service pros from Angi, Thumbtack, Nextdoor, BBB and Werkspot in one run. Paste your links in any mix: each one is detected and scraped. One dataset with rating (1-5), text, date, reply and source. Runs the Apify Actor carabain-digital/home-services-reviews. Use for tasks like: Compare contractor reviews: Angi, Thumbtack, Nextdoor, BBB; Export Angi and Thumbtack reviews to CSV or Excel."
---

# Home Services Reviews Scraper

Scrape customer reviews of contractors and home service pros from Angi, Thumbtack, Nextdoor, BBB and Werkspot in one run. Paste your links in any mix: each one is detected and scraped. One dataset with rating (1-5), text, date, reply and source.

Runs on Apify as the Actor [`carabain-digital/home-services-reviews`](https://apify.com/carabain-digital/home-services-reviews?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,192 reviews with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~home-services-reviews/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~home-services-reviews/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"startUrls": [{"url": "https://www.angi.com/companylist/us/in/indianapolis-indiana/well-done-handyman-and-home-renovations-reviews-451158.htm"}, {"url": "https://www.thumbtack.com/ca/san-francisco/handyman/be-james-team/service/485225739830198305"}, {"url": "https://nextdoor.com/pages/707-plumbers-santa-rosa-ca/"}, {"url": "https://www.bbb.org/us/tx/austin/profile/job-listing-service/indeed-0825-1000101793/customer-reviews"}], "limit": 50}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/home-services-reviews").call(run_input={"startUrls": [{"url": "https://www.angi.com/companylist/us/in/indianapolis-indiana/well-done-handyman-and-home-renovations-reviews-451158.htm"}, {"url": "https://www.thumbtack.com/ca/san-francisco/handyman/be-james-team/service/485225739830198305"}, {"url": "https://nextdoor.com/pages/707-plumbers-santa-rosa-ca/"}, {"url": "https://www.bbb.org/us/tx/austin/profile/job-listing-service/indeed-0825-1000101793/customer-reviews"}], "limit": 50})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/home-services-reviews`.

## Input

| Field | Type | Description |
|---|---|---|
| `startUrls` | array | Links to the contractor or business pages whose reviews you want. Mix sites and add several links per site: the site of each link is detected automatically. Supported: Angi, Thumbtack, Nextdoor, BBB and Werkspot. Links from other sites are skipped with a message in the log. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many reviews for each link (newest first). Ten links with 100 means up to 1,000 reviews. Default: `100`. |
| `cutoff_date` | string | Skip reviews published before this date (YYYY-MM-DD). Useful for incremental runs. |
| `aiAnalysis` | boolean | Adds sentiment (positive, neutral, negative or mixed), a score from -1 to 1, up to 5 topics and a one-line summary to each review with text, using AI. Charged as the AI review analysis add-on per analysed review; reviews without text are not charged. Default: `false`. |

## Examples

**Compare contractor reviews: Angi, Thumbtack, Nextdoor, BBB**: Fetch the 50 newest reviews per link from Angi, Thumbtack, Nextdoor and BBB into a single dataset with a source column. Replace the example links with your contractor's pages on each site to compare ratings, reviews and replies across platforms, then export to CSV, Excel or JSON.

```json
{
  "startUrls": [
    {
      "url": "https://www.angi.com/companylist/us/in/indianapolis-indiana/well-done-handyman-and-home-renovations-reviews-451158.htm"
    },
    {
      "url": "https://www.thumbtack.com/ca/san-francisco/handyman/be-james-team/service/485225739830198305"
    },
    {
      "url": "https://nextdoor.com/pages/707-plumbers-santa-rosa-ca/"
    },
    {
      "url": "https://www.bbb.org/us/tx/austin/profile/job-listing-service/indeed-0825-1000101793/customer-reviews"
    }
  ],
  "limit": 50
}
```

**Export Angi and Thumbtack reviews to CSV or Excel**: Download the 50 newest reviews per link from Angi and Thumbtack in one dataset: star rating, review text, date, reviewer and the site each review came from. Replace the example links with your own, schedule it weekly to monitor new reviews and export to CSV, Excel or JSON.

```json
{
  "startUrls": [
    {
      "url": "https://www.angi.com/companylist/us/in/indianapolis-indiana/well-done-handyman-and-home-renovations-reviews-451158.htm"
    },
    {
      "url": "https://www.thumbtack.com/ca/san-francisco/handyman/be-james-team/service/485225739830198305"
    }
  ],
  "limit": 50
}
```

## Output

One dataset item per result. Main fields: `source`, `published_at`, `rating`, `author`, `title`, `text`, `reply`, `source_url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Review ID prefixed with the site, e.g. angi:abc123; unique across sites and stable across runs |
| `source` | string | Site the review comes from: angi, thumbtack, nextdoor, bbb or werkspot |
| `author` | string | Reviewer name as shown on the site |
| `rating` | integer | Rating as 1-5 stars |
| `title` | string | Review title, if the site has one |
| `text` | string | Review text |
| `published_at` | string | Publication date, ISO 8601 UTC |
| `reply` | string | The business's public reply |
| `replied_at` | string | Reply date, ISO 8601 UTC, if published |
| `source_url` | string | The link from your input that this review was scraped from |

## Price

Pay per result: from $2.28 per 1,000 reviews on the Apify Free plan, less on paid plans, plus $0.025 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- Angi review: $2.28 per 1,000.
- Thumbtack review: $3.12 per 1,000.
- Nextdoor review: $3.36 per 1,000.
- BBB review: $3.60 per 1,000.
- Werkspot review: $2.40 per 1,000.
- AI review analysis: $3.00 per 1,000. Sentiment, a score from -1 to 1, topics and a one-line summary for each review with text.

Store page, reviews and full documentation: [Home Services Reviews Scraper](https://apify.com/carabain-digital/home-services-reviews?fpr=4ijnom)

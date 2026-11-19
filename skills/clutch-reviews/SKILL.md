---
name: clutch-reviews
description: "Scrape all client reviews of any agency or IT company on Clutch.co: rating, project, budget, quality/schedule/cost scores, reviewer and full interview. Export to JSON, CSV or Excel. Runs the Apify Actor carabain-digital/clutch-reviews. Use for tasks like: Export Clutch reviews to CSV or Excel; Monitor new Clutch reviews of your agency."
---

# Clutch Reviews Scraper

Scrape all client reviews of any agency or IT company on Clutch.co: rating, project, budget, quality/schedule/cost scores, reviewer and full interview. Export to JSON, CSV or Excel.

Runs on Apify as the Actor [`carabain-digital/clutch-reviews`](https://apify.com/carabain-digital/clutch-reviews), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,500 reviews with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~clutch-reviews/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~clutch-reviews/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"url": "https://clutch.co/profile/andersen", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/clutch-reviews").call(run_input={"url": "https://clutch.co/profile/andersen", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/clutch-reviews`.

## Input

| Field | Type | Description |
|---|---|---|
| `url` | string | The company's Clutch profile, e.g. https://clutch.co/profile/andersen |
| `limit` | integer | Stop after this many reviews (newest first). Leave empty for all. |
| `cutoff_date` | string | Skip reviews published before this date (YYYY-MM-DD). Useful for incremental runs. |
| `aiAnalysis` | boolean | Adds sentiment (positive, neutral, negative or mixed), a score from -1 to 1, up to 5 topics and a one-line summary to each review with text, using AI. Charged as the AI review analysis add-on per analysed review; reviews without text are not charged. Default: `false`. |

## Examples

**Export Clutch reviews to CSV or Excel**: Download the newest 100 client reviews of an agency or IT company on Clutch: rating, quality, schedule and cost scores, project budget and services, reviewer and the full interview. Replace the example URL with any company's Clutch profile and export the result to CSV, Excel or JSON.

```json
{
  "url": "https://clutch.co/profile/andersen",
  "limit": 100
}
```

**Monitor new Clutch reviews of your agency**: Fetch the 20 newest client reviews of an agency or IT company on Clutch. Schedule this task daily or weekly to track new reviews of your agency or your competitors, send them to Google Sheets, Slack or your own app, and pay only for the reviews you fetch.

```json
{
  "url": "https://clutch.co/profile/andersen",
  "limit": 20
}
```

## Output

One dataset item per result. Main fields: `published_at`, `rating`, `title`, `author`, `reviewer_position`, `project_size`, `services`, `text`, `source_url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Clutch review ID, stable across runs |
| `author` | string | Reviewer name, or Anonymous |
| `rating` | integer | Rating as 1-5 stars |
| `title` | string | Project title of the review |
| `text` | string | Clutch's summary of the project and the client's feedback |
| `published_at` | string | Publication date, ISO 8601 UTC |
| `reply` | string | Always null: Clutch shows no provider replies |
| `replied_at` | string | Always null |
| `quote` | string | The client's one-line quote |
| `rating_exact` | number | Overall rating as given (0.5-5, half stars) |
| `rating_quality` | number | Quality score (service & deliverables), 0.5-5 |
| `rating_schedule` | number | Schedule score (on time / deadlines), 0.5-5 |
| `rating_cost` | number | Cost score (value / within estimates), 0.5-5 |
| `rating_willing_to_refer` | number | Willing-to-refer score (NPS), 0.5-5 |
| `services` | array | Services provided in the project |
| `focus_areas` | array | Focus areas / technologies |
| `project_size` | string | Project budget range, or Confidential |
| `project_length` | string | Project period, e.g. Feb. 2025 - Ongoing |
| `reviewer_position` | string | Reviewer's position and company |
| `reviewer_industry` | string | Client's industry |
| `reviewer_location` | string | Client's location |
| `reviewer_company_size` | string | Client's company size |
| `review_type` | string | Online Review or Phone Interview |
| `interview` | string | The full review interview (background, challenge, solution, results) as text |
| `source_url` | string | The Clutch profile URL that was scraped |

## Price

Pay per result: $2.00 per 1,000 reviews on the Apify Free plan, less on paid plans, plus $0.025 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- AI review analysis: $3.00 per 1,000. Sentiment, a score from -1 to 1, topics and a one-line summary for each review with text.

Store page, reviews and full documentation: [Clutch Reviews Scraper](https://apify.com/carabain-digital/clutch-reviews)

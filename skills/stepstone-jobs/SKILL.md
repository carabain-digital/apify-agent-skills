---
name: stepstone-jobs
description: "Scrape job postings from StepStone.de (Germany): title, company, location, posting date, valid-through, home-office flag and company page. Optional full details: description and employment type. Filter by keyword, location, radius and age. German residential proxy. Export to JSON, CSV or Excel. Runs the Apify Actor carabain-digital/stepstone-jobs. Use for tasks like: Software developer jobs in Berlin (StepStone); Nursing jobs of the last 7 days (StepStone)."
---

# StepStone Jobs Scraper

Scrape job postings from StepStone.de (Germany): title, company, location, posting date, valid-through, home-office flag and company page. Optional full details: description and employment type. Filter by keyword, location, radius and age. German residential proxy. Export to JSON, CSV or Excel.

Runs on Apify as the Actor [`carabain-digital/stepstone-jobs`](https://apify.com/carabain-digital/stepstone-jobs?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 5,263 job postings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~stepstone-jobs/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~stepstone-jobs/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"keyword": "Softwareentwickler", "location": "Berlin", "radius_km": 30, "limit": 50}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/stepstone-jobs").call(run_input={"keyword": "Softwareentwickler", "location": "Berlin", "radius_km": 30, "limit": 50})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/stepstone-jobs`.

## Input

| Field | Type | Description |
|---|---|---|
| `keyword` | string | Job title or skill to search for. Example: `Softwareentwickler` or `Pflegefachkraft`. Give a keyword and/or a location. |
| `location` | string | City or postcode to search around. Example: `Berlin` or `Frankfurt am Main`. Give a keyword and/or a location. |
| `radius_km` | integer | Search radius around the location, in km. Example: `30`. Needs a location. |
| `posted_since_days` | integer | Only jobs posted in the last N days. Example: `7`. StepStone filters 1, 3, 7 and 14 days; other values are applied on the posting date. Leave empty for all ads. |
| `limit` | integer | Stop after this many jobs. Example: `100`. StepStone shows at most 150 jobs per search (6 pages of 25), so the maximum is 150: split bigger needs over several keywords or locations. Default: `100`. |
| `includeDetails` | boolean | Open every job's detail page to add the full description (plain text) and employment type. Adds $1 per 1,000 enriched jobs. Slower: StepStone detail pages are large. Default: `false`. |

## Examples

**Software developer jobs in Berlin (StepStone)**: The newest software developer jobs within 30 km of Berlin on StepStone.de, with company, location, posting date and home-office flag.

```json
{
  "keyword": "Softwareentwickler",
  "location": "Berlin",
  "radius_km": 30,
  "limit": 50
}
```

**Nursing jobs of the last 7 days (StepStone)**: Nursing jobs (Pflegefachkraft) in Germany posted in the last 7 days on StepStone.de, with full description and employment type.

```json
{
  "keyword": "Pflegefachkraft",
  "posted_since_days": 7,
  "limit": 100,
  "includeDetails": true
}
```

## Output

One dataset item per result. Main fields: `posted_at`, `title`, `company`, `city`, `salary_min`, `salary_max`, `remote`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable job id, `<source>:<site id>` |
| `source` | string | Short site name |
| `url` | string | Public job page |
| `title` | string | Job title |
| `company` | string | Employer or recruiter name |
| `company_url` | string | Company page on StepStone |
| `location` | string | Location text as shown on StepStone; ads for several sites list them all, comma separated |
| `city` | string | First location of the ad |
| `postcode` | string | Postcode when the ad has one |
| `region` | string | Region or state when the site gives it |
| `country` | string | ISO country code of the site |
| `lat` | number | Latitude (not provided by this site) |
| `lon` | number | Longitude (not provided by this site) |
| `remote` | string | onsite or hybrid (home office possible) from StepStone's home-office flag; null when the ad has no flag |
| `posted_at` | string | Date StepStone published the ad |
| `valid_through` | string | Date the ad stops being published |
| `employment_type` | array | schema.org employment types, e.g. ["FULL_TIME"] (full-details add-on); empty without it |
| `hours_per_week_min` | number | Minimum hours per week |
| `hours_per_week_max` | number | Maximum hours per week |
| `salary_min` | number | Employer-stated salary only; StepStone's own salary estimates are not copied |
| `salary_max` | number | Employer-stated salary only; StepStone's own salary estimates are not copied |
| `salary_currency` | string | Salary currency, ISO 4217 |
| `salary_period` | string | hour, month or year |
| `education_level` | string | Not provided by StepStone |
| `description` | string | Full job description as plain text (full-details add-on) |
| `apply_url` | string | Not provided by StepStone (use the job url) |
| `scraped_at` | string | When the job was scraped, ISO 8601 UTC |

## Price

Pay per result: $0.95 per 1,000 job postings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $2.00 per 1,000. Opens each job's detail page: full description (plain text), employment type, exact posting and valid-through dates.

Store page, reviews and full documentation: [StepStone Jobs Scraper](https://apify.com/carabain-digital/stepstone-jobs?fpr=4ijnom)

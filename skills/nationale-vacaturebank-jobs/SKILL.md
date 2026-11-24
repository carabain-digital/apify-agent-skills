---
name: nationale-vacaturebank-jobs
description: "Scrape job postings from Nationale Vacaturebank (nationalevacaturebank.nl), the largest Dutch job board: title, company, location with coordinates, salary, hours, contract, education, full description and apply link. Filter by keyword, place, radius and date. Runs the Apify Actor carabain-digital/nationale-vacaturebank-jobs. Use for tasks like: Export Nationale Vacaturebank jobs to CSV or Excel; Monitor new Nationale Vacaturebank jobs daily."
---

# Nationale Vacaturebank Jobs Scraper

Scrape job postings from Nationale Vacaturebank (nationalevacaturebank.nl), the largest Dutch job board: title, company, location with coordinates, salary, hours, contract, education, full description and apply link. Filter by keyword, place, radius and date.

Runs on Apify as the Actor [`carabain-digital/nationale-vacaturebank-jobs`](https://apify.com/carabain-digital/nationale-vacaturebank-jobs?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 10,000 job postings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~nationale-vacaturebank-jobs/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~nationale-vacaturebank-jobs/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"keyword": "developer", "location": "Utrecht", "radius_km": 25, "limit": 500}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/nationale-vacaturebank-jobs").call(run_input={"keyword": "developer", "location": "Utrecht", "radius_km": 25, "limit": 500})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/nationale-vacaturebank-jobs`.

## Input

| Field | Type | Description |
|---|---|---|
| `keyword` | string | Search words in the job title and text, e.g. developer or verpleegkundige. Leave empty for all jobs. |
| `location` | string | Dutch city, town or postcode to search around, e.g. Utrecht or 3511. Leave empty for the whole country. |
| `radius_km` | integer | Search distance around the location in kilometres, e.g. 25. Only used with a location. Default 10. |
| `posted_since_days` | integer | Only jobs published within this many days, e.g. 7. Use 1 for a daily monitoring run. Leave empty for all. |
| `limit` | integer | Stop after this many jobs, newest first, e.g. 100. The site serves at most 10,000 jobs per search: split bigger exports by location or date. Leave empty for everything the search finds (up to 10,000). |

## Examples

**Export Nationale Vacaturebank jobs to CSV or Excel**: Download up to 500 job postings from Nationale Vacaturebank for a keyword and place: title, company, salary, hours, location, description and apply link. Change the keyword and location to your market and export to CSV, Excel or JSON.

```json
{
  "keyword": "developer",
  "location": "Utrecht",
  "radius_km": 25,
  "limit": 500
}
```

**Monitor new Nationale Vacaturebank jobs daily**: Fetch the Nationale Vacaturebank jobs posted in the last 24 hours for a keyword. Schedule this task daily to track new vacancies, feed a job alert, a CRM or a spreadsheet, and pay only for the jobs you fetch.

```json
{
  "keyword": "verpleegkundige",
  "location": "Rotterdam",
  "radius_km": 25,
  "posted_since_days": 1,
  "limit": 100
}
```

## Output

One dataset item per result. Main fields: `posted_at`, `title`, `company`, `location`, `salary_min`, `salary_max`, `salary_period`, `hours_per_week_max`, `employment_type`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable job ID: '<source>:<site UUID>' |
| `source` | string | Source site: nationalevacaturebank or intermediair |
| `url` | string | Public job page on the site |
| `title` | string | Job title |
| `company` | string | Employer or recruiter name |
| `company_url` | string | Company website, when the posting has one |
| `location` | string | Work location as shown on the site |
| `city` | string | City of the work location |
| `postcode` | string | Dutch postcode of the work location |
| `region` | string | Province |
| `country` | string | ISO country code |
| `lat` | number | Latitude of the work location |
| `lon` | number | Longitude of the work location |
| `remote` | string | onsite, hybrid or remote; null when the posting does not say |
| `posted_at` | string | Publication date as the site gives it (ISO 8601; the time is midnight Dutch time, written in UTC) |
| `valid_through` | string | Date the posting expires (ISO 8601) |
| `employment_type` | array | Contract types as the site labels them, in Dutch: Vast, Tijdelijk, Freelance, Stage, Bijbaan, Vrijwilliger |
| `hours_per_week_min` | number | Minimum hours per week |
| `hours_per_week_max` | number | Maximum hours per week |
| `salary_min` | number | Minimum salary in EUR, for the period in salary_period |
| `salary_max` | number | Maximum salary in EUR, for the period in salary_period |
| `salary_currency` | string | Currency of the salary, EUR |
| `salary_period` | string | month, hour or year; null when the site does not say which |
| `education_level` | string | Required education as the site labels it (MBO, HBO, WO, ...) |
| `description` | string | Full job description as plain text |
| `apply_url` | string | Where to apply, usually the employer's own page; null when applying happens on the site |
| `scraped_at` | string | When the job was scraped (UTC, ISO 8601) |

## Price

Pay per result: $0.50 per 1,000 job postings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.

Store page, reviews and full documentation: [Nationale Vacaturebank Jobs Scraper](https://apify.com/carabain-digital/nationale-vacaturebank-jobs?fpr=4ijnom)

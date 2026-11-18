---
name: jobbird-jobs
description: "Scrape job postings from Jobbird.com (Netherlands): title, company, city, posting date, salary, hours and education from the search results. Optional full details: description, apply link, employment type. Filter by keyword, location, radius and age. No proxy needed. Export to JSON, CSV or Excel. Runs the Apify Actor carabain-digital/jobbird-jobs. Use for tasks like: Developer jobs in Utrecht (Jobbird); New Dutch vacancies of the last 7 days (Jobbird)."
---

# Jobbird Jobs Scraper

Scrape job postings from Jobbird.com (Netherlands): title, company, city, posting date, salary, hours and education from the search results. Optional full details: description, apply link, employment type. Filter by keyword, location, radius and age. No proxy needed. Export to JSON, CSV or Excel.

Runs on Apify as the Actor [`carabain-digital/jobbird-jobs`](https://apify.com/carabain-digital/jobbird-jobs), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 10,000 job postings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~jobbird-jobs/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~jobbird-jobs/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"keyword": "developer", "location": "Utrecht", "radius_km": 15, "limit": 50}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/jobbird-jobs").call(run_input={"keyword": "developer", "location": "Utrecht", "radius_km": 15, "limit": 50})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/jobbird-jobs`.

## Input

| Field | Type | Description |
|---|---|---|
| `keyword` | string | What to search for, in the job title or text. Example: `developer` or `verpleegkundige`. Leave empty to list all jobs, newest first. |
| `location` | string | Place name to search around. Example: `Utrecht`. Leave empty for all of the Netherlands. |
| `radius_km` | integer | Search radius around the location, in km. Example: `15`. Needs a location; without a radius Jobbird shows the whole country. |
| `posted_since_days` | integer | Only jobs refreshed in the last N days. Example: `7`. Handy for daily monitoring. Leave empty for all ads. |
| `limit` | integer | Stop after this many jobs, newest first. Example: `100`. Up to 10,000; about 15 jobs per second of runtime. Default: `100`. |
| `includeDetails` | boolean | Open every job's detail page to add the full description, original posting date, apply link, region and a salary range when the ad has one. Adds $1 per 1,000 enriched jobs. Default: `false`. |

## Examples

**Developer jobs in Utrecht (Jobbird)**: The 50 newest developer vacancies within 15 km of Utrecht on Jobbird, with company, city, salary and hours.

```json
{
  "keyword": "developer",
  "location": "Utrecht",
  "radius_km": 15,
  "limit": 50
}
```

**New Dutch vacancies of the last 7 days (Jobbird)**: The 100 newest vacancies from the last 7 days across the Netherlands on Jobbird, with full details (description and apply link).

```json
{
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
| `company` | string | Employer or, for most ads, the recruiter name shown on Jobbird |
| `company_url` | string | Not provided by Jobbird |
| `location` | string | Location text as shown on the site |
| `city` | string | City (first location when the ad lists several) |
| `postcode` | string | Not provided by Jobbird |
| `region` | string | Region from the ad's address (full-details add-on) |
| `country` | string | ISO country code of the site |
| `lat` | number | Latitude (not provided by this site) |
| `lon` | number | Longitude (not provided by this site) |
| `remote` | string | onsite, hybrid or remote when the site says |
| `posted_at` | string | Date the ad was last refreshed by its feed (the date Jobbird sorts on); with full details the original posting date |
| `valid_through` | string | Not provided by Jobbird for most ads |
| `employment_type` | array | schema.org employment types, e.g. ["FULL_TIME"]; empty without the full-details add-on |
| `hours_per_week_min` | number | Minimum hours per week |
| `hours_per_week_max` | number | Maximum hours per week |
| `salary_min` | number | Lowest stated salary |
| `salary_max` | number | Highest stated salary |
| `salary_currency` | string | Salary currency, ISO 4217 |
| `salary_period` | string | hour, month or year; Jobbird does not state it, so it is guessed from the amount |
| `education_level` | string | Education level as shown on Jobbird, e.g. MBO - HBO |
| `description` | string | Full job description as plain text (full-details add-on) |
| `apply_url` | string | Jobbird's tracked redirect to the employer's apply page (full-details add-on) |
| `scraped_at` | string | When the job was scraped, ISO 8601 UTC |

## Price

Pay per result: $0.50 per 1,000 job postings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. Opens each job's detail page: full description (plain text), original posting date, apply link, region and salary range when stated.

Store page, reviews and full documentation: [Jobbird Jobs Scraper](https://apify.com/carabain-digital/jobbird-jobs)

---
name: germany-jobs
description: "Search Arbeitsagentur and StepStone at once for Stellenangebote in Germany. One keyword, one dataset: merged, deduplicated across both boards, newest first. Optional full descriptions. Residential proxy included. Export to JSON, CSV or Excel. Runs the Apify Actor carabain-digital/germany-jobs. Use for tasks like: Pflegefachkraft jobs in Berlin from two boards; Softwareentwickler jobs of the last 7 days."
---

# Germany Jobs Scraper

Search Arbeitsagentur and StepStone at once for Stellenangebote in Germany. One keyword, one dataset: merged, deduplicated across both boards, newest first. Optional full descriptions. Residential proxy included. Export to JSON, CSV or Excel.

Runs on Apify as the Actor [`carabain-digital/germany-jobs`](https://apify.com/carabain-digital/germany-jobs?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 6,666 job postings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~germany-jobs/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~germany-jobs/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"keyword": "Pflegefachkraft", "location": "Berlin", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/germany-jobs").call(run_input={"keyword": "Pflegefachkraft", "location": "Berlin", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/germany-jobs`.

## Input

| Field | Type | Description |
|---|---|---|
| `keyword` | string | Job title or skill, searched on both sites. German terms work best. Example: `Pflegefachkraft` or `Softwareentwickler`. StepStone needs a keyword and/or a location; leave both empty only to browse all of Arbeitsagentur. |
| `location` | string | German city or postcode, passed to both sites. Example: `Berlin` or `20095`. Leave empty to search all of Germany. |
| `radius_km` | integer | Search radius around the location, in km. Example: `25`. Needs a location. |
| `posted_since_days` | integer | Only jobs posted in the last N days. Example: `7`. Leave empty for all. |
| `limit` | integer | Maximum number of jobs in the dataset, newest first, after merging and removing duplicates. Example: `200`. Each site is also capped by itself: StepStone 150 per search, Arbeitsagentur 10,000. Default: `100`. |
| `limit_per_site` | integer | Optional. Fetch at most this many jobs from each site, so one big site cannot crowd out the rest. Example: `30`. Default: the value of Max jobs. |
| `sites` | array | Which sites to search. Default: all of them. Example: `["arbeitsagentur", "stepstone"]`. Default: `["arbeitsagentur", "stepstone"]`. |
| `includeDetails` | boolean | Open each job's detail page on Arbeitsagentur and StepStone to add the full description and more. Adds $1 per 1,000 enriched jobs. Slower. Default: `false`. |

## Examples

**Pflegefachkraft jobs in Berlin from two boards**: The newest Pflegefachkraft Stellenangebote in Berlin from Arbeitsagentur and StepStone in one deduplicated list.

```json
{
  "keyword": "Pflegefachkraft",
  "location": "Berlin",
  "limit": 100
}
```

**Softwareentwickler jobs of the last 7 days**: Softwareentwickler jobs posted in the last 7 days on Arbeitsagentur and StepStone, newest first, with full descriptions.

```json
{
  "keyword": "Softwareentwickler",
  "posted_since_days": 7,
  "limit": 150,
  "includeDetails": true
}
```

## Output

One dataset item per result. Main fields: `posted_at`, `source`, `title`, `company`, `city`, `country`, `also_on`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable job id, `<source>:<site id>` |
| `source` | string | Site the item was taken from: arbeitsagentur, stepstone |
| `url` | string | Public job page |
| `title` | string | Job title |
| `company` | string | Employer or recruiter name |
| `company_url` | string | Company page when the site has one |
| `location` | string | Location text as shown on the site |
| `city` | string | City (used for cross-site dedup) |
| `postcode` | string | Postcode when the ad has one |
| `region` | string | Region or state when the site gives it |
| `country` | string | ISO country code of the site (NL, BE or DE) |
| `lat` | number | Latitude when the site gives it |
| `lon` | number | Longitude when the site gives it |
| `remote` | string | onsite, hybrid or remote when the site says so |
| `posted_at` | string | Date or time the site published the job (the list is sorted on it, newest first) |
| `valid_through` | string | Date the ad stops being published, when the site gives it |
| `employment_type` | array | Employment type(s) as the site gives them |
| `hours_per_week_min` | number | Minimum hours per week |
| `hours_per_week_max` | number | Maximum hours per week |
| `salary_min` | number | Employer-stated minimum salary |
| `salary_max` | number | Employer-stated maximum salary |
| `salary_currency` | string | Salary currency, ISO 4217 |
| `salary_period` | string | hour, month or year |
| `education_level` | string | Education level when the site gives it |
| `description` | string | Full job description as plain text (always for Nationale Vacaturebank and Intermediair, otherwise with the full-details add-on) |
| `apply_url` | string | Apply link when the site gives it |
| `scraped_at` | string | When the job was scraped, ISO 8601 UTC |
| `also_on` | array | Other sites that carry the same job (same title, company and city); empty when only one does |

## Price

Pay per result: $0.75 per 1,000 job postings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. Opens each job's detail page on Arbeitsagentur and StepStone: full description and extra fields.

Store page, reviews and full documentation: [Germany Jobs Scraper](https://apify.com/carabain-digital/germany-jobs?fpr=4ijnom)

---
name: vdab-jobs
description: "Scrape job postings from VDAB, the Flemish employment service and Belgium's largest job board: title, employer, municipality, contract type, experience and full description. Filter by keyword, location, radius and date. Fast, no proxy needed. Runs the Apify Actor carabain-digital/vdab-jobs. Use for tasks like: verpleegkundige jobs in Gent: newest VDAB listings; boekhouder jobs in Antwerpen: newest VDAB listings."
---

# VDAB Jobs Scraper

Scrape job postings from VDAB, the Flemish employment service and Belgium's largest job board: title, employer, municipality, contract type, experience and full description. Filter by keyword, location, radius and date. Fast, no proxy needed.

Runs on Apify as the Actor [`carabain-digital/vdab-jobs`](https://apify.com/carabain-digital/vdab-jobs?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 5,263 job postings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~vdab-jobs/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~vdab-jobs/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"keyword": "verpleegkundige", "location": "Gent", "radius_km": 25, "posted_since_days": 7, "limit": 50, "includeDetails": false}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/vdab-jobs").call(run_input={"keyword": "verpleegkundige", "location": "Gent", "radius_km": 25, "posted_since_days": 7, "limit": 50, "includeDetails": false})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/vdab-jobs`.

## Input

| Field | Type | Description |
|---|---|---|
| `keyword` | string | Job title or search term. Dutch terms work best. Example: `verpleegkundige`, `boekhouder`. Leave empty to browse all jobs. |
| `location` | string | Flemish municipality or postcode. Example: `Gent` or `9000`. Leave empty for all of Flanders and Brussels. If you give no radius, 10 km is used. |
| `radius_km` | integer | Search radius around the location in kilometres. Example: `20`. Needs a location. Defaults to 10 km when a location is set. |
| `posted_since_days` | integer | Only jobs published in the last N days. Example: `7`. Leave empty for all. |
| `limit` | integer | Stop after this many jobs (newest first). Example: `100`. Up to 10,000. Default: `100`. |
| `includeDetails` | boolean | Also fetch each job's detail: the full job description (plain text, including profile and offer), postcode, region, required education, the apply link and the Belgian company number. Adds $1.00 per 1,000 jobs on top of the base price. One extra request per job. Default: `false`. |

## Examples

**verpleegkundige jobs in Gent: newest VDAB listings**: Scrape the newest VDAB (vdab.be) job postings for verpleegkundige in Gent from the last 7 days: title, employer, location, contract type and salary when published. Export to JSON, CSV or Excel.

```json
{
  "keyword": "verpleegkundige",
  "location": "Gent",
  "radius_km": 25,
  "posted_since_days": 7,
  "limit": 50,
  "includeDetails": false
}
```

**boekhouder jobs in Antwerpen: newest VDAB listings**: Scrape the newest VDAB (vdab.be) job postings for boekhouder in Antwerpen from the last 14 days: title, employer, location, contract type and salary when published. Export to JSON, CSV or Excel.

```json
{
  "keyword": "boekhouder",
  "location": "Antwerpen",
  "radius_km": 25,
  "posted_since_days": 14,
  "limit": 50,
  "includeDetails": false
}
```

## Output

One dataset item per result. Main fields: `posted_at`, `title`, `company`, `city`, `contract_type`, `salary_min`, `salary_max`, `salary_period`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable job ID, `<source>:<site id>` |
| `source` | string | Short site name |
| `url` | string | Public job URL on the site |
| `title` | string | Job title |
| `company` | string | Employer (the end client, not a staffing agency that posts for them, where the site says so) |
| `company_url` | string | Employer website (the site does not publish it, so null) |
| `location` | string | Job location as shown on the site |
| `city` | string | City / municipality |
| `postcode` | string | Postal code |
| `region` | string | State (Bundesland) or region |
| `country` | string | ISO country code |
| `lat` | number | Latitude |
| `lon` | number | Longitude |
| `remote` | string | onsite, hybrid or remote |
| `posted_at` | string | Date or time the job was first published, ISO 8601 |
| `modified_at` | string | Date or time the posting was last changed, ISO 8601 |
| `valid_through` | string | Closing date (not published by this site, null) |
| `employment_type` | array | schema.org employment types: FULL_TIME, PART_TIME, TEMPORARY, CONTRACTOR |
| `contract_type` | string | Mapped from VDAB's contract line |
| `hours_per_week_min` | number | Minimum hours per week (not published by this site, null) |
| `hours_per_week_max` | number | Maximum hours per week (not published by this site, null) |
| `salary_min` | number | Lower end of the advertised salary |
| `salary_max` | number | Upper end of the advertised salary |
| `salary_currency` | string | ISO currency code |
| `salary_period` | string | hour, month or year |
| `education_level` | string | Required education, as the site words it |
| `description` | string | Full job description in plain text. Only filled with the Full details add-on |
| `apply_url` | string | Link to apply, when the site gives one |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `experience_level` | string | Required experience as VDAB words it, e.g. Minstens 2 jaar ervaring |
| `company_type` | string | direct_employer, recruiter (staffing agency) or unknown (third-party feed such as Jobat) |
| `company_id` | string | Belgian company number (KBO/ondernemingsnummer), only with the Full details add-on and when published |

## Price

Pay per result: $0.95 per 1,000 job postings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One job enriched with its detail page: the full job description (plain text, including profile and offer), postcode, region, required education, the apply link and the Belgian company number.

Store page, reviews and full documentation: [VDAB Jobs Scraper](https://apify.com/carabain-digital/vdab-jobs?fpr=4ijnom)

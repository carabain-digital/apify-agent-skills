---
name: welcometothejungle-jobs
description: "Scrape job postings from Welcome to the Jungle (welcometothejungle.com) in France: title, company, city, contract (CDI, alternance, stage), salary, remote policy and description. Filter by keyword, city, radius, contract and date. Fast, no proxy needed. Runs the Apify Actor carabain-digital/welcometothejungle-jobs. Use for tasks like: Développeur jobs in Paris: newest Welcome to the Jungle offers; Alternance jobs in Lyon: Welcome to the Jungle offers."
---

# Welcome to the Jungle Jobs Scraper

Scrape job postings from Welcome to the Jungle (welcometothejungle.com) in France: title, company, city, contract (CDI, alternance, stage), salary, remote policy and description. Filter by keyword, city, radius, contract and date. Fast, no proxy needed.

Runs on Apify as the Actor [`carabain-digital/welcometothejungle-jobs`](https://apify.com/carabain-digital/welcometothejungle-jobs), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 17,857 job postings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~welcometothejungle-jobs/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~welcometothejungle-jobs/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"keyword": "développeur", "location": "Paris", "radius_km": 15, "posted_since_days": 7, "limit": 50, "includeDetails": false}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/welcometothejungle-jobs").call(run_input={"keyword": "développeur", "location": "Paris", "radius_km": 15, "posted_since_days": 7, "limit": 50, "includeDetails": false})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/welcometothejungle-jobs`.

## Input

| Field | Type | Description |
|---|---|---|
| `keyword` | string | Job title or search term, French or English. Example: `développeur`, `product manager`, `alternance marketing`. Leave empty to browse all jobs. |
| `location` | string | French city, department, region or postcode. Example: `Lyon`, `Rhône`, `Île-de-France` or `69003`. Leave empty for all of France. Jobs abroad are never returned. |
| `radius_km` | integer | Search radius around the location in kilometres. Example: `25`. Needs a location (a French city or postcode); we search around the city centre. A postcode without a radius uses 10 km. |
| `posted_since_days` | integer | Only jobs published in the last N days. Example: `7`. Leave empty for all. |
| `wttj_contract_types` | array | Only these contract types. Example: `alternance` and `stage` for students. Leave empty for all types. |
| `limit` | integer | Stop after this many jobs (newest first). Example: `100`. Up to 10,000. Default: `100`. |
| `includeDetails` | boolean | Also fetch each job's detail: the full job description and profile (plain text), the apply link of the employer's own application system, company website, postcode and coordinates, skills, tools and start date. Adds $1.00 per 1,000 jobs on top of the base price. One extra request per job. Default: `false`. |

## Examples

**Développeur jobs in Paris: newest Welcome to the Jungle offers**: Scrape the newest Welcome to the Jungle (welcometothejungle.com) job postings for développeur in Paris and 15 km around, from the last 7 days: title, company, contract type, salary and remote policy when published. Export to JSON, CSV or Excel.

```json
{
  "keyword": "développeur",
  "location": "Paris",
  "radius_km": 15,
  "posted_since_days": 7,
  "limit": 50,
  "includeDetails": false
}
```

**Alternance jobs in Lyon: Welcome to the Jungle offers**: Scrape Welcome to the Jungle (welcometothejungle.com) alternance and stage job postings in Lyon and 25 km around from the last 14 days: title, company, contract, duration, salary and remote policy when published. Export to JSON, CSV or Excel.

```json
{
  "location": "Lyon",
  "radius_km": 25,
  "posted_since_days": 14,
  "wttj_contract_types": [
    "apprenticeship",
    "internship"
  ],
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
| `company_url` | string | Employer website. Only filled with the Full details add-on |
| `location` | string | City of the job (the first French office, or the searched city, for jobs with several offices) |
| `city` | string | City / municipality |
| `postcode` | string | Postal code. Only filled with the Full details add-on |
| `region` | string | Region, as the site shows it |
| `country` | string | ISO country code (FR) |
| `lat` | number | Latitude (null for jobs with several offices unless the Full details add-on is on) |
| `lon` | number | Longitude (null for jobs with several offices unless the Full details add-on is on) |
| `remote` | string | onsite, hybrid (partial or occasional remote) or remote (full remote); null when the employer did not say |
| `posted_at` | string | Date and time the job was published, ISO 8601 |
| `valid_through` | string | Closing date (not published by this site, null) |
| `employment_type` | array | Contract type as Welcome to the Jungle labels it, as a list with one entry: CDI, CDD / Temporaire, Stage, Alternance, Freelance, Temps partiel, VIE, Autre |
| `contract_type` | string | permanent, fixed_term, freelance, student or other (null for part-time and unknown) |
| `hours_per_week_min` | number | Minimum hours per week (not published by this site, null) |
| `hours_per_week_max` | number | Maximum hours per week (not published by this site, null) |
| `salary_min` | number | Lower end of the advertised salary |
| `salary_max` | number | Upper end of the advertised salary |
| `salary_currency` | string | ISO currency code |
| `salary_period` | string | hour, month or year |
| `education_level` | string | Required education, e.g. Bac +5 (when published) |
| `description` | string | Job description in plain text: summary, missions and profile from the search. With the Full details add-on: the complete job description and profile |
| `apply_url` | string | Apply link of the employer's application system. Only with the Full details add-on, when the job has one |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `locations` | array | All office cities of the job (jobs can have several) |
| `contract_duration_months_min` | number | Minimum contract duration in months (fixed-term, internship, alternance), when published |
| `contract_duration_months_max` | number | Maximum contract duration in months, when published |
| `experience_years_min` | number | Minimum years of experience asked, when published |
| `category` | string | Occupation group, e.g. Développeur de logiciels |
| `sectors` | array | Industry sectors of the employer |
| `benefits` | array | Benefits listed by the employer, as shown on the site |
| `language` | string | Language of the job text (fr, en, ...) |
| `start_date` | string | Planned start date. Only with the Full details add-on, when present |
| `skills` | array | Skills listed for the job. Only with the Full details add-on |
| `tools` | array | Tools listed for the job. Only with the Full details add-on |

## Price

Pay per result: $0.28 per 1,000 job postings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One job enriched with its detail page: the full job description and profile (plain text), the apply link of the employer's application system, company website, postcode, coordinates, skills, tools and start date.

Store page, reviews and full documentation: [Welcome to the Jungle Jobs Scraper](https://apify.com/carabain-digital/welcometothejungle-jobs)

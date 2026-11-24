---
name: arbeitsagentur-jobs
description: "Scrape job postings from the German Arbeitsagentur Jobbörse (Bundesagentur für Arbeit): title, employer, location with coordinates, salary, contract type and full description. Filter by keyword, city, radius and date. Fast, no proxy needed. Runs the Apify Actor carabain-digital/arbeitsagentur-jobs. Use for tasks like: Pflegefachkraft jobs in Hamburg: newest Arbeitsagentur listings; Softwareentwickler jobs in Berlin: newest Arbeitsagentur listin."
---

# Arbeitsagentur Jobs Scraper

Scrape job postings from the German Arbeitsagentur Jobbörse (Bundesagentur für Arbeit): title, employer, location with coordinates, salary, contract type and full description. Filter by keyword, city, radius and date. Fast, no proxy needed.

Runs on Apify as the Actor [`carabain-digital/arbeitsagentur-jobs`](https://apify.com/carabain-digital/arbeitsagentur-jobs?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 8,333 job postings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~arbeitsagentur-jobs/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~arbeitsagentur-jobs/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"keyword": "Pflegefachkraft", "location": "Hamburg", "radius_km": 25, "posted_since_days": 7, "limit": 50, "includeDetails": false}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/arbeitsagentur-jobs").call(run_input={"keyword": "Pflegefachkraft", "location": "Hamburg", "radius_km": 25, "posted_since_days": 7, "limit": 50, "includeDetails": false})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/arbeitsagentur-jobs`.

## Input

| Field | Type | Description |
|---|---|---|
| `keyword` | string | Job title or search term. German terms work best. Example: `Pflegefachkraft`, `Softwareentwickler`. Leave empty to browse all jobs. |
| `location` | string | City or postcode in Germany. Example: `Hamburg` or `20095`. Leave empty for all of Germany. |
| `radius_km` | integer | Search radius around the location in kilometres. Example: `25`. Needs a location. The Jobbörse accepts 5, 10, 15, 25, 50, 100 or 200 km; other values are rounded by the site. |
| `posted_since_days` | integer | Only jobs published in the last N days. Example: `7`. The site filters in steps of 1, 7, 14 and 28 days; we then cut to the exact number of days. Leave empty for all. |
| `limit` | integer | Stop after this many jobs (newest first). Example: `100`. Up to 10,000. Default: `100`. |
| `includeDetails` | boolean | Also fetch each job's detail: the full job description (plain text), the apply link when the job is posted by an external employer site, and whether the poster is a recruiter or a direct employer. Adds $1.00 per 1,000 jobs on top of the base price. One extra request per job. Default: `false`. |

## Examples

**Pflegefachkraft jobs in Hamburg: newest Arbeitsagentur listings**: Scrape the newest Arbeitsagentur (arbeitsagentur.de) job postings for Pflegefachkraft in Hamburg from the last 7 days: title, employer, location, contract type and salary when published. Export to JSON, CSV or Excel.

```json
{
  "keyword": "Pflegefachkraft",
  "location": "Hamburg",
  "radius_km": 25,
  "posted_since_days": 7,
  "limit": 50,
  "includeDetails": false
}
```

**Softwareentwickler jobs in Berlin: newest Arbeitsagentur listin**: Scrape the newest Arbeitsagentur (arbeitsagentur.de) job postings for Softwareentwickler in Berlin from the last 14 days: title, employer, location, contract type and salary when published. Export to JSON, CSV or Excel.

```json
{
  "keyword": "Softwareentwickler",
  "location": "Berlin",
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
| `contract_type` | string | permanent, fixed_term, temp_agency, freelance, student or other |
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
| `category` | string | Occupation (Hauptberuf), e.g. Krankenschwester/-pfleger |
| `distance_km` | integer | Distance to the searched location in km, when a location was given |
| `street` | string | Street and house number, when published |
| `company_type` | string | recruiter or direct_employer. Only filled with the Full details add-on |
| `partner` | string | Partner board that syndicated the job (e.g. XING). Only with the Full details add-on, when present |

## Price

Pay per result: $0.60 per 1,000 job postings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One job enriched with its detail page: the full job description (plain text), the apply link when the job is posted by an external employer site, and whether the poster is a recruiter or a direct employer.

Store page, reviews and full documentation: [Arbeitsagentur Jobs Scraper](https://apify.com/carabain-digital/arbeitsagentur-jobs?fpr=4ijnom)

---
name: france-jobs
description: "Search Welcome to the Jungle and France Travail at once for offres d'emploi in France. One keyword, one dataset: merged, deduplicated across both boards, newest first. CDI, CDD, alternance. Optional full descriptions. No proxy needed. JSON, CSV or Excel. Runs the Apify Actor carabain-digital/france-jobs. Use for tasks like: Développeur jobs in Lyon from two boards; Alternance marketing offers of the last 7 days."
---

# France Jobs Scraper

Search Welcome to the Jungle and France Travail at once for offres d'emploi in France. One keyword, one dataset: merged, deduplicated across both boards, newest first. CDI, CDD, alternance. Optional full descriptions. No proxy needed. JSON, CSV or Excel.

Runs on Apify as the Actor [`carabain-digital/france-jobs`](https://apify.com/carabain-digital/france-jobs), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 6,666 job postings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~france-jobs/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~france-jobs/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"keyword": "développeur", "location": "Lyon", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/france-jobs").call(run_input={"keyword": "développeur", "location": "Lyon", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/france-jobs`.

## Input

| Field | Type | Description |
|---|---|---|
| `keyword` | string | Job title or skill, searched on both sites. French terms work best. Example: `développeur`, `infirmier` or `alternance marketing`. France Travail needs a keyword and/or a location; leave both empty only to browse all of Welcome to the Jungle. |
| `location` | string | French city, department or postcode, sent to both sites. Example: `Lyon`, `Rhône` or `69003`. Leave empty to search all of France. |
| `radius_km` | integer | Search radius around the location, in km. Example: `25`. Needs a location. |
| `posted_since_days` | integer | Only jobs posted in the last N days. Example: `7`. France Travail dates are whole days. Leave empty for all. |
| `limit` | integer | Maximum number of jobs in the dataset, newest first, after merging and removing duplicates. Example: `200`. France Travail shows at most 1,150 jobs per search; Welcome to the Jungle up to 10,000. Default: `100`. |
| `limit_per_site` | integer | Optional. Fetch at most this many jobs from each site, so one big site cannot crowd out the rest. Example: `30`. Default: the value of Max jobs. |
| `sites` | array | Which sites to search. Default: all of them. Example: `["welcometothejungle", "francetravail"]`. Default: `["welcometothejungle", "francetravail"]`. |
| `includeDetails` | boolean | Open each job's detail page on Welcome to the Jungle and France Travail to add the full description, salary, skills and more. Adds $1 per 1,000 enriched jobs. Slower. Default: `false`. |

## Examples

**Développeur jobs in Lyon from two boards**: The newest développeur offres d'emploi in Lyon from Welcome to the Jungle and France Travail in one deduplicated list.

```json
{
  "keyword": "développeur",
  "location": "Lyon",
  "limit": 100
}
```

**Alternance marketing offers of the last 7 days**: Alternance marketing offers posted in the last 7 days on Welcome to the Jungle and France Travail, newest first, with full descriptions.

```json
{
  "keyword": "alternance marketing",
  "posted_since_days": 7,
  "limit": 150,
  "includeDetails": true
}
```

## Output

One dataset item per result. Main fields: `posted_at`, `source`, `title`, `company`, `city`, `also_on`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable job ID, `<source>:<site id>` |
| `source` | string | Site the item was taken from: welcometothejungle, francetravail |
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
| `posted_at` | string | When the job was published (the list is sorted on it, newest first). Welcome to the Jungle: ISO 8601 date and time. France Travail: a day only (YYYY-MM-DD) |
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
| `description` | string | Full job description as plain text. Only with the Full details add-on |
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
| `modified_at` | string | Date the offer was last changed, ISO 8601. Only filled with the Full details add-on |
| `department` | string | Department number of the job, e.g. 69 |
| `contract_duration_months` | integer | Contract duration in months, when stated. Only filled with the Full details add-on |
| `salary_text` | string | Salary as France Travail words it, e.g. 'Mensuel de 2000.0 Euros à 2400.0 Euros'. Only filled with the Full details add-on |
| `experience` | string | Experience asked, e.g. '1 An(s)' or 'Débutant accepté'. Only filled with the Full details add-on |
| `rome_code` | string | ROME occupation code, e.g. D1102. Only filled with the Full details add-on |
| `sector` | string | Industry sector of the employer. Only filled with the Full details add-on, when present |
| `origin` | string | France Travail only: where the offer comes from, France Travail itself or a partner board (HELLOWORK, METEOJOB, CADREMPLOI, ...). About 80% are partner reposts |
| `also_on` | array | Other sites that carry the same job (same title, company and city); empty when only one does |
| `details_fetched` | boolean | True when the Full details add-on enriched this item |

## Price

Pay per result: $0.75 per 1,000 job postings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. Opens each job's detail page on Welcome to the Jungle and France Travail: full description, salary, skills and extra fields.

Store page, reviews and full documentation: [France Jobs Scraper](https://apify.com/carabain-digital/france-jobs)

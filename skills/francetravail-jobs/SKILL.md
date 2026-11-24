---
name: francetravail-jobs
description: "Scrape job postings from France Travail (ex Pôle emploi, candidat.francetravail.fr): title, employer, city, contract (CDI, CDD, intérim, alternance), date and partner board; salary, ROME code and full text with the add-on. Filter by keyword, city, radius, contract. Runs the Apify Actor carabain-digital/francetravail-jobs. Use for tasks like: Infirmier jobs in Paris: newest France Travail offers; Alternance jobs in Toulouse: France Travail offers."
---

# France Travail Jobs Scraper

Scrape job postings from France Travail (ex Pôle emploi, candidat.francetravail.fr): title, employer, city, contract (CDI, CDD, intérim, alternance), date and partner board; salary, ROME code and full text with the add-on. Filter by keyword, city, radius, contract.

Runs on Apify as the Actor [`carabain-digital/francetravail-jobs`](https://apify.com/carabain-digital/francetravail-jobs?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 10,000 job postings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~francetravail-jobs/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~francetravail-jobs/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"keyword": "infirmier", "location": "Paris", "radius_km": 20, "posted_since_days": 7, "limit": 100, "includeDetails": false}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/francetravail-jobs").call(run_input={"keyword": "infirmier", "location": "Paris", "radius_km": 20, "posted_since_days": 7, "limit": 100, "includeDetails": false})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/francetravail-jobs`.

## Input

| Field | Type | Description |
|---|---|---|
| `keyword` | string | Job title or search term, French. Example: `vendeur`, `infirmier`, `développeur web`. Several words narrow the search. Needs a keyword or a location. |
| `location` | string | French city, postcode, department number or region. Example: `Lyon`, `69003`, `69` or `Occitanie`. Leave empty for all of France (then a keyword is required). |
| `radius_km` | integer | Search radius around the city in kilometres, 0 to 100. Example: `25`. Needs a location. France Travail's own default is 10 km. |
| `posted_since_days` | integer | Only jobs published in the last N days. Example: `7`. The site filters in steps of 1, 3, 7, 14 and 31 days; we then cut to the exact number of days. Leave empty for all. |
| `ft_contract_types` | array | Only these contract types. Example: `Apprentissage` for alternance. Leave empty for all types. |
| `ft_partner_offers` | boolean | Also return offers that France Travail republishes from partner job boards (HelloWork, Meteojob, Cadremploi, ...). They make up most of the list. Switch off to get only France Travail's own offers. Default: `true`. |
| `limit` | integer | Stop after this many jobs (newest first). Example: `100`. France Travail shows at most 1,150 jobs per search. Default: `100`. |
| `includeDetails` | boolean | Also fetch each job's detail: the full job description, salary (min, max, period and original text), ROME occupation code, experience and training asked, skills, working hours, company website and last-modified date. Adds $1.00 per 1,000 jobs on top of the base price. One extra request per job. Default: `false`. |

## Examples

**Infirmier jobs in Paris: newest France Travail offers**: Scrape the newest France Travail (ex Pôle emploi) job postings for infirmier in Paris and 20 km around from the last 7 days: title, employer, city, contract type, publication date and partner board. Export to JSON, CSV or Excel.

```json
{
  "keyword": "infirmier",
  "location": "Paris",
  "radius_km": 20,
  "posted_since_days": 7,
  "limit": 100,
  "includeDetails": false
}
```

**Alternance jobs in Toulouse: France Travail offers**: Scrape France Travail (ex Pôle emploi) apprenticeship (contrat d'apprentissage) job postings in Toulouse and 30 km around from the last 14 days: title, employer, city, contract and publication date. Export to JSON, CSV or Excel.

```json
{
  "location": "Toulouse",
  "radius_km": 30,
  "posted_since_days": 14,
  "ft_contract_types": [
    "E2"
  ],
  "limit": 100,
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
| `company` | string | Employer name; null when the employer is anonymous (about 15% of offers) |
| `company_url` | string | Employer website. Only filled with the Full details add-on, when the employer gave one |
| `location` | string | City or district of the job as France Travail shows it, e.g. LYON 09 |
| `city` | string | City or district of the job |
| `postcode` | string | Postal code (not published in search results, null) |
| `region` | string | Region (not published, null) |
| `country` | string | ISO country code |
| `lat` | number | Latitude (not published, null) |
| `lon` | number | Longitude (not published, null) |
| `remote` | string | onsite, hybrid or remote (not published, null) |
| `posted_at` | string | Day the offer was published, YYYY-MM-DD (Paris time). France Travail only shows 'today', 'yesterday', 'N days ago'; offers older than 30 days show a lower bound |
| `valid_through` | string | Closing date (not published, null) |
| `employment_type` | array | Contract and working time as France Travail labels them, as a list: contract (CDI, CDD, Intérim, Saisonnier, Contrat apprentissage, Profession libérale, ...) and, when known, Temps plein or Temps partiel |
| `contract_type` | string | permanent, fixed_term, temp_agency, freelance, student or other |
| `hours_per_week_min` | number | Hours per week, when stated. Only filled with the Full details add-on |
| `hours_per_week_max` | number | Hours per week, when stated. Only filled with the Full details add-on |
| `salary_min` | number | Lower end of the advertised salary. Only filled with the Full details add-on |
| `salary_max` | number | Upper end of the advertised salary. Only filled with the Full details add-on |
| `salary_currency` | string | ISO currency code (EUR). Only filled with the Full details add-on |
| `salary_period` | string | hour, month or year. Only filled with the Full details add-on |
| `education_level` | string | Training asked, as France Travail words it. Only filled with the Full details add-on |
| `description` | string | Job description. In search results only the first 200 characters; with the Full details add-on the complete text |
| `apply_url` | string | Not published (you apply on the job URL), null |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `modified_at` | string | Date the offer was last changed, ISO 8601. Only filled with the Full details add-on |
| `department` | string | Department number of the job, e.g. 69 |
| `contract_duration_months` | integer | Contract duration in months, when stated. Only filled with the Full details add-on |
| `salary_text` | string | Salary as France Travail words it, e.g. 'Mensuel de 2000.0 Euros à 2400.0 Euros'. Only filled with the Full details add-on |
| `experience` | string | Experience asked, e.g. '1 An(s)' or 'Débutant accepté'. Only filled with the Full details add-on |
| `category` | string | Occupation (ROME appellation), e.g. Boulanger / Boulangère. Only filled with the Full details add-on |
| `rome_code` | string | ROME occupation code, e.g. D1102. Only filled with the Full details add-on |
| `sector` | string | Industry sector of the employer. Only filled with the Full details add-on, when present |
| `skills` | array | Skills listed for the job. Only filled with the Full details add-on |
| `origin` | string | Where the offer comes from: France Travail itself or a partner board (HELLOWORK, METEOJOB, CADREMPLOI, ...) |

## Price

Pay per result: $0.50 per 1,000 job postings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One job enriched with its detail: the full job description, salary (parsed and as text), ROME occupation, experience, training, skills, company website, working hours and last-modified date.

Store page, reviews and full documentation: [France Travail Jobs Scraper](https://apify.com/carabain-digital/francetravail-jobs?fpr=4ijnom)

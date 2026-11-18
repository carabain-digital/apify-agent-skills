---
name: bbb-scraper
description: "Scrape BBB (Better Business Bureau, bbb.org) business listings by keyword and location: name, BBB rating, accreditation, phone, address, categories. Add-ons: owners and contacts, email, website, business details, customer reviews and complaints. Runs the Apify Actor carabain-digital/bbb-scraper. Use for tasks like: BBB roofing contractor leads in Denver with owners; Monitor BBB complaints and reviews of a company."
---

# BBB Scraper - Leads, Ratings, Reviews, Complaints

Scrape BBB (Better Business Bureau, bbb.org) business listings by keyword and location: name, BBB rating, accreditation, phone, address, categories. Add-ons: owners and contacts, email, website, business details, customer reviews and complaints.

Runs on Apify as the Actor [`carabain-digital/bbb-scraper`](https://apify.com/carabain-digital/bbb-scraper), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,000 businesss with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~bbb-scraper/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~bbb-scraper/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"searchTerms": ["roofing"], "locations": ["Denver, CO"], "accreditedOnly": true, "includeDetails": true, "maxItems": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/bbb-scraper").call(run_input={"searchTerms": ["roofing"], "locations": ["Denver, CO"], "accreditedOnly": true, "includeDetails": true, "maxItems": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/bbb-scraper`.

## Input

| Field | Type | Description |
|---|---|---|
| `searchTerms` | array | What to search for, as on bbb.org: a trade or category (`plumber`, `roofing`, `dentist`) or a business name. Each term is searched in every location below. Example: `roofing`. |
| `locations` | array | Where to search: `City, ST`, a ZIP code or a state code. Leave empty to search the whole country. Example: `Denver, CO`, `78753` or `TX`. |
| `country` | string | BBB covers the United States and Canada. Example: `US`. Values: `US`, `CA`. Default: `"US"`. |
| `startUrls` | array | BBB search result pages (`https://www.bbb.org/search?find_text=...&find_loc=...`, filters in the URL are kept) or business profiles (`https://www.bbb.org/us/tx/austin/profile/.../indeed-0825-1000101793`). A profile URL always comes with its details. Used in addition to the search terms. Format: `[{"url": "https://..."}]`. |
| `maxItems` | integer | Stop after this many businesses in total. Example: `100`. Default: `100`. |
| `maxItemsPerSearch` | integer | Cap each search term x location, so one big city does not use up Max businesses. Empty = no cap. Example: `200`. |
| `splitLargeSearches` | boolean | BBB shows at most 225 results per search. When a search has more, the Actor repeats it per category (up to about 2,500 businesses per search). For more, add ZIP codes or nearby towns as locations. Default: `true`. |
| `accreditedOnly` | boolean | Only BBB Accredited Businesses. Default: `false`. |
| `minRating` | string | Skip businesses with a lower BBB letter grade (and unrated ones). Example: `A`. Values: `A+`, `A`, `A-`, `B+`, `B`, `B-`, `C+`, `C`, `C-`, `D+`, `D`, `D-`, `F`. |
| `maxDistanceMiles` | integer | Only businesses within this many miles of each location. Example: `10`. |
| `sort` | string | Order of the search results, as on bbb.org. Values: `relevance`, `distance`, `rating`, `name`, `name_desc`. Default: `"relevance"`. |
| `includeDetails` | boolean | Open each business profile for website, email, owners and managers with their titles, extra phones, entity type, years in business, start, incorporation and accreditation dates, review and complaint counts and the description. Adds $2.00 per 1,000 businesses. Default: `false`. |
| `maxReviews` | integer | Also collect up to this many customer reviews per business, newest first: rating, text, date, author and the business's reply. 0 = off. Adds $1.50 per 1,000 reviews. Example: `20`. Default: `0`. |
| `maxComplaints` | integer | Also collect up to this many BBB complaints per business, newest first: type, status, date, text and the replies of business and customer. 0 = off. Adds $1.50 per 1,000 complaints. Example: `20`. Default: `0`. |

## Examples

**BBB roofing contractor leads in Denver with owners**: BBB Accredited roofing contractors in Denver, CO from bbb.org with BBB rating, phone, address, website, email and the owners' names and titles. Swap the trade and city for any market. Export to JSON, CSV or Excel.

```json
{
  "searchTerms": [
    "roofing"
  ],
  "locations": [
    "Denver, CO"
  ],
  "accreditedOnly": true,
  "includeDetails": true,
  "maxItems": 100
}
```

**Monitor BBB complaints and reviews of a company**: The newest BBB complaints and customer reviews of a company (here Indeed) with status, text and the company's replies, plus its BBB rating and counts. Schedule it daily or weekly to catch new complaints. Export to JSON, CSV or Excel.

```json
{
  "startUrls": [
    {
      "url": "https://www.bbb.org/us/tx/austin/profile/job-listing-service/indeed-0825-1000101793"
    }
  ],
  "maxReviews": 20,
  "maxComplaints": 20
}
```

## Output

One dataset item per result. Main fields: `name`, `bbbRating`, `accredited`, `category`, `phone`, `city`, `state`, `website`, `reviewCount`, `complaintCount`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable BBB business ID, `<bbbId>-<businessId>` |
| `name` | string | Business name |
| `url` | string | BBB profile URL |
| `bbbRating` | string | BBB letter grade, A+ to F (null if not rated) |
| `ratingScore` | number | BBB's numeric rating score behind the grade (search results only) |
| `accredited` | boolean | BBB Accredited Business |
| `category` | string | Primary category |
| `categories` | array | All categories |
| `phone` | string | Main phone number |
| `address` | string | Street address |
| `city` | string | City |
| `state` | string | State or province code |
| `postalCode` | string | ZIP or postal code |
| `latitude` | number | Latitude |
| `longitude` | number | Longitude |
| `serviceAreas` | array | Areas the business serves |
| `logoUrl` | string | Logo image URL |
| `localBbb` | string | Local BBB office that keeps the file |
| `requestQuoteUrl` | string | BBB request-a-quote link, if enabled |
| `outOfBusiness` | boolean | Marked out of business |
| `searchTerm` | string | Search term that found the business |
| `searchLocation` | string | Search location that found the business |
| `scrapedAt` | string | Scrape time, ISO 8601 UTC |
| `website` | string | Website (add-on Details) |
| `email` | string | Email address, de-obfuscated (add-on Details) |
| `contacts` | array | Owners and managers: name, title, isPrincipal (add-on Details) |
| `additionalPhones` | array | Other phone numbers (add-on Details) |
| `additionalEmails` | array | Other email addresses (add-on Details) |
| `additionalWebsites` | array | Other websites and social links (add-on Details) |
| `alternateNames` | array | Other business names (add-on Details) |
| `description` | string | Business description (add-on Details) |
| `entityType` | string | Legal entity type, e.g. Corporation or LLC (add-on Details) |
| `yearsInBusiness` | integer | Years in business (add-on Details) |
| `numberOfEmployees` | integer | Number of employees, if given (add-on Details) |
| `businessStarted` | string | Business start date (add-on Details) |
| `incorporated` | string | Incorporation date (add-on Details) |
| `accreditedSince` | string | BBB accreditation date (add-on Details) |
| `bbbFileOpened` | string | Date BBB opened the file (add-on Details) |
| `ratingReasons` | array | Reasons BBB gives for the grade (add-on Details) |
| `reviewCount` | integer | Number of customer reviews on BBB (add-on Details) |
| `averageReviewRating` | number | Average customer review rating, 1-5 (add-on Details) |
| `complaintCount` | integer | Number of complaints on BBB (add-on Details) |
| `complaintsClosedLast3Years` | integer | Complaints closed in the past 3 years (add-on Details) |
| `complaintsClosedLast12Months` | integer | Complaints closed in the past 12 months (add-on Details) |
| `operatingHours` | object | Opening hours, if listed (add-on Details) |
| `additionalInfo` | object | Other profile texts, e.g. Products and Services (add-on Details) |
| `reviews` | array | Customer reviews: id, author, rating, text, date, reply, repliedAt (add-on Reviews) |
| `complaints` | array | Complaints: id, type, status, date, text, responses (add-on Complaints) |

## Price

Pay per result: $2.50 per 1,000 businesss on the Apify Free plan, less on paid plans, plus $0.025 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- Details (owners, email, website): $2.00 per 1,000. One business enriched from its BBB profile: website, email, owners and managers, extra phones, entity type, years in business, key dates, review and complaint counts, description.
- Customer review: $1.50 per 1,000. One BBB customer review of a business: rating, text, date, author and the business's reply.
- Complaint: $1.50 per 1,000. One BBB complaint against a business: type, status, date, text and the replies of business and customer.

Store page, reviews and full documentation: [BBB Scraper - Leads, Ratings, Reviews, Complaints](https://apify.com/carabain-digital/bbb-scraper)

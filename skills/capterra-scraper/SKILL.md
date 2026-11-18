---
name: capterra-scraper
description: "Scrape Capterra (capterra.com) software products per category: name, vendor, ratings, review counts, features. Add-ons: pricing plans, AI pros and cons, integrations, alternatives, and user reviews. Runs the Apify Actor carabain-digital/capterra-scraper. Use for tasks like: Capterra CRM software with pricing plans; Monitor Capterra reviews of competing products."
---

# Capterra Scraper - Software, Pricing, Reviews

Scrape Capterra (capterra.com) software products per category: name, vendor, ratings, review counts, features. Add-ons: pricing plans, AI pros and cons, integrations, alternatives, and user reviews.

Runs on Apify as the Actor [`carabain-digital/capterra-scraper`](https://apify.com/carabain-digital/capterra-scraper), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,000 products with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~capterra-scraper/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~capterra-scraper/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"categories": ["CRM"], "includeDetails": true, "maxItems": 200}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/capterra-scraper").call(run_input={"categories": ["CRM"], "includeDetails": true, "maxItems": 200})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/capterra-scraper`.

## Input

| Field | Type | Description |
|---|---|---|
| `categories` | array | Capterra categories by name as on capterra.com/categories (`CRM`, `Project Management`, `Payroll`), by the slug from the category URL (`customer-relationship-management-software`) or by the full URL. Every product of each category is collected, in Capterra's default order. |
| `startUrls` | array | Category pages (`https://www.capterra.com/payroll-software/`) or product pages (`https://www.capterra.com/p/61368/Salesforce/`, a reviews page URL works too). A product URL always comes with its details. Used in addition to the categories. Format: `[{"url": "https://..."}]`. |
| `maxItems` | integer | Stop after this many products in total. Example: `100`. Default: `100`. |
| `maxItemsPerCategory` | integer | Cap each category, so one big category does not use up Max products. Empty = no cap. Example: `50` for the top 50 of each. |
| `minRating` | string | Skip products rated lower (and unrated ones). Example: `4`. Values: `3`, `3.5`, `4`, `4.5`. |
| `minReviews` | integer | Skip products with fewer Capterra reviews. Example: `10`. |
| `includeDetails` | boolean | Open each product page for the starting price and every pricing plan, deployment, support and training options, review sentiment, Capterra's AI summary of pros and cons, reviewer industries, company sizes and job functions, all categories, Shortlist awards, integrations, alternatives and the matching G2 page. Adds $2.00 per 1,000 products. Default: `false`. |
| `maxReviews` | integer | Also collect up to this many Capterra user reviews per product, newest first: overall and sub-ratings, title, text, pros, cons, date, reviewer role, industry and company size, the product they switched from, and the vendor's reply. 0 = off. Adds $1.00 per 1,000 reviews. Example: `20`. Default: `0`. |

## Examples

**Capterra CRM software with pricing plans**: All CRM products listed on Capterra with vendor, ratings, review counts, starting price, every pricing plan, AI pros and cons and integrations. Swap the category for any software market. Export to JSON, CSV or Excel.

```json
{
  "categories": [
    "CRM"
  ],
  "includeDetails": true,
  "maxItems": 200
}
```

**Monitor Capterra reviews of competing products**: The newest Capterra user reviews of a set of products (here Salesforce Sales Cloud, HubSpot CRM and Pipedrive) with ratings, pros, cons and vendor replies. Schedule it weekly to follow what users say. Export to JSON, CSV or Excel.

```json
{
  "startUrls": [
    {
      "url": "https://www.capterra.com/p/61368/Salesforce/"
    },
    {
      "url": "https://www.capterra.com/p/152373/HubSpot-CRM/"
    },
    {
      "url": "https://www.capterra.com/p/132666/Pipedrive/"
    }
  ],
  "maxReviews": 30
}
```

## Output

One dataset item per result. Main fields: `name`, `vendor`, `rating`, `reviewCount`, `startingPrice`, `category`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Capterra product ID (the number in the product URL) |
| `uuid` | string | Capterra product UUID |
| `name` | string | Product name |
| `slug` | string | Product slug in the Capterra URL |
| `url` | string | Capterra product page |
| `reviewsUrl` | string | Capterra reviews page |
| `vendor` | string | Vendor name |
| `rating` | number | Average overall rating, 0-5 |
| `reviewCount` | integer | Number of Capterra reviews |
| `easeOfUseRating` | number | Average ease-of-use rating, 0-5 |
| `customerServiceRating` | number | Average customer service rating, 0-5 |
| `featuresRating` | number | Average features (functionality) rating, 0-5 |
| `valueForMoneyRating` | number | Average value-for-money rating, 0-5 |
| `ratingBreakdown` | object | Number of reviews per star ("1" to "5") |
| `reviewsLast6Months` | integer | Reviews written in the last 6 months |
| `reviewsLast12Months` | integer | Reviews written in the last 12 months |
| `hasFreeTrial` | boolean | A free trial is offered |
| `hasFreeVersion` | boolean | A free version is offered |
| `shortDescription` | string | One-line description |
| `description` | string | Full description |
| `features` | array | Features the product offers |
| `logoUrl` | string | Product logo URL |
| `category` | string | Category the product was listed in |
| `categoryUrl` | string | Category page URL |
| `position` | integer | Rank in the category (Capterra's default order) |
| `scrapedAt` | string | Scrape time, ISO 8601 UTC |
| `startingPrice` | number | Lowest listed price (add-on Details) |
| `startingPriceUnit` | string | What the starting price covers, e.g. Per User, Per Month (add-on Details) |
| `priceCurrency` | string | Currency of the prices (add-on Details) |
| `creditCardRequired` | boolean | The free trial needs a credit card (add-on Details) |
| `pricingPlans` | array | Pricing plans: name, price, unit, description, features (add-on Details) |
| `deployment` | array | Deployment options, e.g. Cloud, SaaS, Web-Based (add-on Details) |
| `support` | array | Support options, e.g. Phone Support, Chat (add-on Details) |
| `training` | array | Training options (add-on Details) |
| `targetCompanySizes` | array | Company sizes the product targets (add-on Details) |
| `sentiment` | object | Share of positive, neutral and negative reviews in percent (add-on Details) |
| `prosSummary` | array | What reviewers like, Capterra's AI summary with share and review count (add-on Details) |
| `consSummary` | array | What reviewers dislike, Capterra's AI summary with share and review count (add-on Details) |
| `topIndustries` | array | Industries of the reviewers, in percent (add-on Details) |
| `companySizes` | array | Company sizes of the reviewers, in percent (add-on Details) |
| `jobFunctions` | array | Job functions of the reviewers, in percent (add-on Details) |
| `categories` | array | All Capterra categories of the product (add-on Details) |
| `awards` | array | Capterra Shortlist awards, e.g. CRM 2026 (add-on Details) |
| `integrations` | array | Integrations: name, Capterra URL (add-on Details) |
| `alternatives` | array | Alternatives Capterra suggests: name, URL, rating, review count (add-on Details) |
| `g2Url` | string | The same product on G2 (add-on Details) |
| `reviews` | array | User reviews: rating, title, text, pros, cons, sub-ratings, date, reviewer, vendor reply (add-on Reviews) |

## Price

Pay per result: $2.50 per 1,000 products on the Apify Free plan, less on paid plans, plus $0.0125 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- Details (pricing plans, pros and cons): $2.00 per 1,000. One product enriched from its Capterra page: pricing plans, deployment and support, AI pros and cons, reviewer profile, categories, integrations, alternatives.
- User review: $1.00 per 1,000. One Capterra user review of a product: ratings, title, text, pros, cons, date, reviewer and the vendor's reply.

Store page, reviews and full documentation: [Capterra Scraper - Software, Pricing, Reviews](https://apify.com/carabain-digital/capterra-scraper)

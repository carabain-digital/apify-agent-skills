---
name: g2-scraper
description: "Scrape G2 (g2.com) software products per category: name, vendor, rating, review count, pros and cons, prices. Add-ons: website, description, pricing plans, languages, and user reviews. Runs the Apify Actor carabain-digital/g2-scraper. Use for tasks like: G2 CRM software vendors with websites and pricing; Monitor G2 reviews of competing products."
---

# G2 Scraper - Software Products, Ratings, Reviews

Scrape G2 (g2.com) software products per category: name, vendor, rating, review count, pros and cons, prices. Add-ons: website, description, pricing plans, languages, and user reviews.

Runs on Apify as the Actor [`carabain-digital/g2-scraper`](https://apify.com/carabain-digital/g2-scraper?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,000 products with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~g2-scraper/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~g2-scraper/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"categories": ["crm"], "includeDetails": true, "maxItems": 200}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/g2-scraper").call(run_input={"categories": ["crm"], "includeDetails": true, "maxItems": 200})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/g2-scraper`.

## Input

| Field | Type | Description |
|---|---|---|
| `categories` | array | G2 categories: the slug from the category URL (`crm` for g2.com/categories/crm), the full URL, or the name (`Project Management Software`). Every product of each category is collected, in G2's default order. Example: `crm`, `project-management`, `email-marketing`. |
| `startUrls` | array | Category pages (`https://www.g2.com/categories/crm`, filters in the URL are kept) or product pages (`https://www.g2.com/products/attio/reviews`). A product URL always comes with its details. Used in addition to the categories. Format: `[{"url": "https://..."}]`. |
| `maxItems` | integer | Stop after this many products in total. Example: `100`. Default: `100`. |
| `maxItemsPerCategory` | integer | Cap each category, so one big category does not use up Max products. Empty = no cap. Example: `50` for the top 50 of each. |
| `minRating` | string | Skip products rated lower (and unrated ones). Example: `4`. Values: `3`, `3.5`, `4`, `4.5`. |
| `minReviews` | integer | Skip products with fewer G2 reviews. Example: `10`. |
| `includeDetails` | boolean | Open each product page for the website, description, all categories, every pricing plan with its price, supported languages, solution type and vendor ID. Adds $2.00 per 1,000 products. Default: `false`. |
| `maxReviews` | integer | Also collect up to this many user reviews per product, newest first: rating, title, full text (likes, dislikes, problems solved), date, author and the vendor's reply. 0 = off. Adds $2.00 per 1,000 reviews. Example: `20`. Default: `0`. |

## Examples

**G2 CRM software vendors with websites and pricing**: All CRM products listed on G2 with vendor, rating, review count, pros and cons, website and every pricing plan. Swap the category for any software market. Export to JSON, CSV or Excel.

```json
{
  "categories": [
    "crm"
  ],
  "includeDetails": true,
  "maxItems": 200
}
```

**Monitor G2 reviews of competing products**: The newest G2 user reviews of a set of products (here Attio, Pipedrive and HubSpot Sales Hub) with rating, full text and vendor replies, plus each product's rating and pricing. Schedule it weekly to follow what users say. Export to JSON, CSV or Excel.

```json
{
  "startUrls": [
    {
      "url": "https://www.g2.com/products/attio/reviews"
    },
    {
      "url": "https://www.g2.com/products/pipedrive/reviews"
    },
    {
      "url": "https://www.g2.com/products/hubspot-sales-hub/reviews"
    }
  ],
  "maxReviews": 30
}
```

## Output

One dataset item per result. Main fields: `name`, `vendor`, `rating`, `reviewCount`, `lowestPrice`, `category`, `website`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | G2 product ID (the slug when G2 shows none) |
| `slug` | string | Product slug in the G2 URL |
| `url` | string | G2 product page |
| `name` | string | Product name |
| `rating` | number | Average star rating, 0-5 |
| `reviewCount` | integer | Number of G2 reviews |
| `pros` | array | What users like most, G2's summary |
| `cons` | array | What users like least, G2's summary |
| `lowestPrice` | number | Lowest listed plan price |
| `highestPrice` | number | Highest listed plan price |
| `priceCurrency` | string | Currency of the prices |
| `pricingPlanCount` | integer | Number of listed pricing plans |
| `vendor` | string | Vendor (seller) name |
| `vendorUrl` | string | Vendor page on G2 |
| `imageUrl` | string | Product logo URL |
| `category` | string | Category the product was listed in |
| `categoryUrl` | string | Category page URL |
| `position` | integer | Rank in the category on its page (G2's default order) |
| `productUuid` | string | G2 product UUID |
| `scrapedAt` | string | Scrape time, ISO 8601 UTC |
| `website` | string | Product website (add-on Details) |
| `description` | string | Product description (add-on Details) |
| `categories` | array | All G2 categories of the product (add-on Details) |
| `pricingPlans` | array | Pricing plans: name, price, currency (add-on Details) |
| `languages` | array | Supported languages (add-on Details) |
| `solutionType` | string | Solution type, e.g. All-in-One or Point Solution (add-on Details) |
| `vendorId` | integer | G2 vendor ID (add-on Details) |
| `reviews` | array | User reviews: id, author, rating, title, text, date, reply (add-on Reviews) |

## Price

Pay per result: $2.50 per 1,000 products on the Apify Free plan, less on paid plans, plus $0.0125 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- Details (website, pricing plans): $2.00 per 1,000. One product enriched from its G2 page: website, description, all categories, pricing plans with prices, languages, solution type.
- User review: $2.00 per 1,000. One G2 user review of a product: rating, title, full text, date, author and the vendor's reply.

Store page, reviews and full documentation: [G2 Scraper - Software Products, Ratings, Reviews](https://apify.com/carabain-digital/g2-scraper?fpr=4ijnom)

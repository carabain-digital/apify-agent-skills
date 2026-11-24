---
name: bienici-listings
description: "Scrape property listings from Bien'ici (bienici.com), the French real-estate portal: apartments, houses, land, parking and commercial property for sale and for rent. Filter by city, price, rooms and area. Newest first, with DPE label. Add-on: agency contact with phone. Runs the Apify Actor carabain-digital/bienici-listings. Use for tasks like: Apartments for rent in Paris: newest on Bien'ici; Houses for sale in Bordeaux with price history on Bien'ici."
---

# Bien'ici Scraper - French Property Listings

Scrape property listings from Bien'ici (bienici.com), the French real-estate portal: apartments, houses, land, parking and commercial property for sale and for rent. Filter by city, price, rooms and area. Newest first, with DPE label. Add-on: agency contact with phone.

Runs on Apify as the Actor [`carabain-digital/bienici-listings`](https://apify.com/carabain-digital/bienici-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,631 listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~bienici-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~bienici-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"location": "Paris", "listing_type": "rent", "property_type": "apartment", "max_price": 1800, "limit": 100, "includeDetails": false}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/bienici-listings").call(run_input={"location": "Paris", "listing_type": "rent", "property_type": "apartment", "max_price": 1800, "limit": 100, "includeDetails": false})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/bienici-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `location` | string | French city, arrondissement, department or postcode. Several values separated by commas. Example: `Lyon`, `Paris 15e`, `69003` or `Rhône, Isère`. Leave empty for all of France. Ignored when you give search URLs. |
| `listing_type` | string | `sale` for properties to buy, `rent` for rentals. Example: `rent`. Values: `sale`, `rent`. Default: `"sale"`. |
| `property_type` | string | Type of property. `any` = everything Bien'ici lists (apartments, houses, land, parking, commercial, new-build programmes). `house` includes townhouses and castles, `apartment` includes lofts. Example: `apartment`. Values: `any`, `house`, `apartment`, `land`, `commercial`, `parking`. Default: `"any"`. |
| `min_price` | integer | Lowest price in euro (rent: per month, charges excluded as on the site). Example: `200000`, for rent `600`. |
| `max_price` | integer | Highest price in euro (rent: per month). Example: `450000`, for rent `1200`. |
| `min_rooms` | integer | Minimum number of rooms (pièces, a studio counts 1). Example: `3`. |
| `min_living_area_m2` | integer | Minimum living area in square metres. Listings without a known area are left out when this is set. Example: `80`. |
| `start_urls` | array | Search result pages copied from bienici.com; they replace the filters above. Example: `https://www.bienici.com/recherche/achat/lyon-69000/appartement?prix-max=400000&pieces-min=3`. Price, surface and rooms filters in the URL are kept. Leave empty to use the filters above. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `100`. Bien'ici returns at most 2,500 results per search: use a price range, a postcode or several search URLs to go deeper. Default: `100`. |
| `includeDetails` | boolean | Also ask Bien'ici for each listing's contact block: agency name, phone number and website. The description, photos, DPE label and dates are already in the base data. Adds $1.00 per 1,000 listings on top of the base price. One extra request per listing, so runs take longer. Default: `false`. |
| `trackHistory` | boolean | Keep the price and status of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our real-estate Actors use `realestate-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `gent-apartments`. Default: `"realestate-history"`. |

## Examples

**Apartments for rent in Paris: newest on Bien'ici**: Scrape the newest apartments for rent in Paris from Bien'ici (bienici.com): monthly rent and charges, arrondissement, rooms, living area, DPE label and agency. Export to JSON, CSV or Excel.

```json
{
  "location": "Paris",
  "listing_type": "rent",
  "property_type": "apartment",
  "max_price": 1800,
  "limit": 100,
  "includeDetails": false
}
```

**Houses for sale in Bordeaux with price history on Bien'ici**: Scrape houses for sale in Bordeaux from Bien'ici (bienici.com) and track their price over time: asking price, price changes, rooms, living area, plot size and DPE. Run it daily to catch price drops.

```json
{
  "location": "Bordeaux",
  "listing_type": "sale",
  "property_type": "house",
  "limit": 200,
  "includeDetails": false,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `title`, `city`, `price`, `living_area_m2`, `rooms`, `status`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `<source>:<site id>` |
| `source` | string | Short site name |
| `url` | string | Public listing URL on the site |
| `title` | string | Listing title as the agent wrote it (not every listing has one) |
| `listing_type` | string | sale or rent |
| `property_type` | string | house, apartment, land, commercial, parking, room or other |
| `property_subtype` | string | The site's own type: flat, house, loft, townhouse, castle, terrain, parking, shop, office, premises, building, programme |
| `status` | string | available, under_offer, sold or rented |
| `price` | number | Asking price in EUR (rent per month for rentals). New-build programmes: lowest price of the programme |
| `price_currency` | string | ISO currency code (EUR) |
| `price_period` | string | month for rentals, null for sales |
| `price_type` | string | asking, on_request, auction (public/online sale), life_annuity or from (new-build project, minimum price) |
| `address` | string | Street, number, postcode and city on one line |
| `street` | string | Street name. Always null: Bien'ici hides the street and house number in its listing data |
| `house_number` | string | House number. Always null, see street |
| `postcode` | string | Belgian postcode (4 digits) |
| `city` | string | City / municipality |
| `region` | string | Region. Always null: Bien'ici does not give one |
| `country` | string | ISO country code (BE) |
| `lat` | number | Latitude. Approximate: Bien'ici blurs most listing positions (250 m disk or the whole city) |
| `lon` | number | Longitude. Approximate, see lat |
| `living_area_m2` | number | Living area in m2 |
| `plot_area_m2` | number | Plot area in m2 |
| `rooms` | integer | Total number of rooms, when the site gives it |
| `bedrooms` | integer | Number of bedrooms |
| `bathrooms` | integer | Number of bathrooms |
| `energy_label` | string | EPC energy label (A++ to G) |
| `construction_year` | integer | Year of construction |
| `listed_at` | string | When the listing was published on Bien'ici: ISO 8601. Null when the import has no date |
| `updated_at` | string | When the listing was last changed on the site, ISO 8601 |
| `agent_name` | string | Agency or seller name. Not every listing shows one; the Full details add-on fills it from the contact block |
| `agent_phone` | string | Agency phone number. Only with the Full details add-on |
| `agent_url` | string | Agency website. Only with the Full details add-on, when the agency has one |
| `image_urls` | array | Photo URLs |
| `description` | string | Listing description |
| `old_price` | number | Always null on Bien'ici (it only flags price decreases, without the old price) |
| `monthly_costs` | number | Rentals: monthly charges on top of the rent, when given |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: true when the status changed since the previous run |
| `price_history` | array | Price history add-on: price and status points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $1.90 per 1,000 listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One listing enriched with its contact block: agency name, phone number and website.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [Bien'ici Scraper - French Property Listings](https://apify.com/carabain-digital/bienici-listings?fpr=4ijnom)

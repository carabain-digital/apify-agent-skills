---
name: zimmo-listings
description: "Scrape real-estate listings from Zimmo (zimmo.be), Belgium: houses, apartments, land and commercial property for sale and for rent. Filter by city or postcode, price, bedrooms, living area and type. Price, address, EPC, agent, photos and the site's own price history. Fast, no proxy needed. Runs the Apify Actor carabain-digital/zimmo-listings. Use for tasks like: Apartments for rent in Antwerp: newest on Zimmo; Houses for sale in Gent with price history on Zimmo."
---

# Zimmo Listings Scraper

Scrape real-estate listings from Zimmo (zimmo.be), Belgium: houses, apartments, land and commercial property for sale and for rent. Filter by city or postcode, price, bedrooms, living area and type. Price, address, EPC, agent, photos and the site's own price history. Fast, no proxy needed.

Runs on Apify as the Actor [`carabain-digital/zimmo-listings`](https://apify.com/carabain-digital/zimmo-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 10,000 listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~zimmo-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~zimmo-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"location": "Antwerpen", "listing_type": "rent", "property_type": "apartment", "max_price": 1500, "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/zimmo-listings").call(run_input={"location": "Antwerpen", "listing_type": "rent", "property_type": "apartment", "max_price": 1500, "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/zimmo-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `location` | string | Belgian city, municipality or postcode. Several values separated by commas (all postcodes or all names). Example: `Gent`, `Antwerpen` or `9000, 9050`. Leave empty for all of Belgium. Ignored when you give search URLs. |
| `listing_type` | string | `sale` for properties to buy, `rent` for rentals. Example: `rent`. Values: `sale`, `rent`. Default: `"sale"`. |
| `property_type` | string | Type of property. `any` = all types. Example: `apartment`. Values: `any`, `house`, `apartment`, `land`, `commercial`, `parking`, `room`. Default: `"any"`. |
| `min_price` | integer | Lowest price in euro (rent: per month). Example: `200000`, for rent `800`. |
| `max_price` | integer | Highest price in euro (rent: per month). Example: `450000`, for rent `1500`. |
| `min_rooms` | integer | Minimum number of bedrooms. Example: `2`. |
| `min_living_area_m2` | integer | Minimum living area in square metres. Listings without a known area are left out when this is set. Example: `80`. |
| `start_urls` | array | Search result pages copied from zimmo.be; they replace the filters above. Example: `https://www.zimmo.be/nl/gent/te-koop/appartement` or an advanced-search URL (`/nl/uitgebreid-zoeken?search=...`). Leave empty to use the filters above. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `100`. Zimmo returns at most about 9,950 results per search: use a place or price range to go deeper. Default: `100`. |
| `trackHistory` | boolean | Keep the price and status of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our real-estate Actors use `realestate-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `gent-apartments`. Default: `"realestate-history"`. |

## Examples

**Apartments for rent in Antwerp: newest on Zimmo**: Scrape the newest apartments for rent in Antwerp from Zimmo (zimmo.be): monthly rent, address, bedrooms, living area, EPC label, agent and photos. Export to JSON, CSV or Excel.

```json
{
  "location": "Antwerpen",
  "listing_type": "rent",
  "property_type": "apartment",
  "max_price": 1500,
  "limit": 100
}
```

**Houses for sale in Gent with price history on Zimmo**: Scrape houses for sale in Gent from Zimmo (zimmo.be) and track their price over time: asking price, previous price, price changes, address, living area and EPC. Run it daily to catch price drops.

```json
{
  "location": "Gent",
  "listing_type": "sale",
  "property_type": "house",
  "limit": 200,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `title`, `city`, `price`, `living_area_m2`, `bedrooms`, `status`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `<source>:<site id>` |
| `source` | string | Short site name |
| `url` | string | Public listing URL on the site |
| `title` | string | Listing title as the agent wrote it (not every listing has one) |
| `listing_type` | string | sale or rent |
| `property_type` | string | house, apartment, land, commercial, parking, room or other |
| `property_subtype` | string | The site's own subtype, e.g. penthouse, duplex, villa |
| `status` | string | available, under_offer, sold or rented |
| `price` | number | Asking price in EUR (sale) or rent per month (rent). Null when the price is on request |
| `price_currency` | string | ISO currency code (EUR) |
| `price_period` | string | month for rentals, null for sales |
| `price_type` | string | asking, on_request, auction (public/online sale), life_annuity or from (new-build project, minimum price) |
| `address` | string | Street, number, postcode and city on one line |
| `street` | string | Street name |
| `house_number` | string | House number including box/addition |
| `postcode` | string | Belgian postcode (4 digits) |
| `city` | string | City / municipality |
| `region` | string | Province, e.g. East Flanders |
| `country` | string | ISO country code (BE) |
| `lat` | number | Latitude |
| `lon` | number | Longitude |
| `living_area_m2` | number | Living area in m2 |
| `plot_area_m2` | number | Plot area in m2 |
| `rooms` | integer | Total number of rooms, when the site gives it |
| `bedrooms` | integer | Number of bedrooms |
| `bathrooms` | integer | Number of bathrooms |
| `energy_label` | string | EPC energy label (A++ to G) |
| `construction_year` | integer | Year of construction |
| `listed_at` | string | When the listing was first published, ISO 8601 |
| `updated_at` | string | When the listing was last changed on the site, ISO 8601 |
| `agent_name` | string | Estate agent, notary or developer. Null for private sellers |
| `agent_phone` | string | Agent phone number |
| `agent_url` | string | Agent website |
| `image_urls` | array | Photo URLs |
| `description` | string | Listing description |
| `site_price_history` | array | Price changes as Zimmo shows them, `[{date, price_before, price_after}]`, oldest first |
| `price_drop_date` | string | When the last price change happened, ISO 8601 (null if the price never changed) |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: true when the status changed since the previous run |
| `price_history` | array | Price history add-on: price and status points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $0.50 per 1,000 listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [Zimmo Listings Scraper](https://apify.com/carabain-digital/zimmo-listings?fpr=4ijnom)

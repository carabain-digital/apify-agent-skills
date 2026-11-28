---
name: immovlan-listings
description: "Scrape property listings from Immovlan.be, the Belgian real-estate portal: price, type, city, EPC label, price-update flag and status for sale and rent. Optional full details (street, coordinates, area, bedrooms, description, agent) and price history. No proxy needed. Runs the Apify Actor carabain-digital/immovlan-listings. Use for tasks like: Apartments for sale in Gent (Immovlan); Rentals in Brussels under 1,500 euro (Immovlan)."
---

# Immovlan Listings Scraper

Scrape property listings from Immovlan.be, the Belgian real-estate portal: price, type, city, EPC label, price-update flag and status for sale and rent. Optional full details (street, coordinates, area, bedrooms, description, agent) and price history. No proxy needed.

Runs on Apify as the Actor [`carabain-digital/immovlan-listings`](https://apify.com/carabain-digital/immovlan-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 10,000 listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~immovlan-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~immovlan-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"location": "Gent", "listing_type": "sale", "property_type": "apartment", "limit": 50, "includeDetails": false, "trackHistory": false}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/immovlan-listings").call(run_input={"location": "Gent", "listing_type": "sale", "property_type": "apartment", "limit": 50, "includeDetails": false, "trackHistory": false})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/immovlan-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `location` | string | Belgian municipality, postcode, province or region. Example: `Gent`, `9000` or `Brussels`; several separated by commas, e.g. `8500, 8790, Zulte`. A municipality includes its suburbs, like on the site. Leave empty for all of Belgium. |
| `listing_type` | string | `sale` for properties for sale, `rent` for rentals. Example: `sale`. Values: `sale`, `rent`. Default: `"sale"`. |
| `property_type` | string | Only this kind of property. Example: `apartment`. Leave empty for all types. Values: `house`, `apartment`, `land`, `commercial`, `parking`. |
| `min_price` | integer | Minimum price in euro (monthly rent for rentals). Example: `200000`. |
| `max_price` | integer | Maximum price in euro (monthly rent for rentals). Example: `450000`. |
| `min_rooms` | integer | Minimum number of bedrooms. Immovlan does not publish a total room count, so `min_rooms` filters on bedrooms. Example: `2`. |
| `min_living_area_m2` | integer | Minimum living area in square metres. Example: `80`. |
| `start_urls` | array | Optional. Search result URLs copied from immovlan.be (any language), for filters this form does not have, e.g. `https://immovlan.be/en/real-estate?transactiontypes=for-sale&propertytypes=house&towns=9000-gent`. When set, the other search fields are ignored. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings. The site shows at most 1,000 results per search; above 1,000 the Actor splits the search by price band. Example: `100`. Up to 10,000. Default: `100`. |
| `includeDetails` | boolean | Also open each listing's detail page: street and house number, coordinates, living and plot area, bedrooms, bathrooms, construction year (when given), full description, listing date, photos and the agency with phone when published. Adds $1.00 per 1,000 listings on top of the base price. One extra request per listing; new-build project cards have no such page and are not charged. Default: `false`. |
| `trackHistory` | boolean | Remember each listing's price and status in a named storage in your own Apify account and add `previous_price`, `price_change`, `price_change_pct`, `status_changed`, `first_seen_at` and `price_history` to every listing you have seen before. Schedule the Actor to build a series. Adds $1.00 per 1,000 listings that already had history (the first sighting is free). Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that keeps the history (lowercase letters, digits and dashes). The same name across runs and across our other real-estate Actors shares one history. Example: `realestate-history`. Default: `"realestate-history"`. |

## Examples

**Apartments for sale in Gent (Immovlan)**: Scrape the newest apartments for sale in Gent from Immovlan.be: price, city, EPC label, status and price-update flag. Export to JSON, CSV or Excel.

```json
{
  "location": "Gent",
  "listing_type": "sale",
  "property_type": "apartment",
  "limit": 50,
  "includeDetails": false,
  "trackHistory": false
}
```

**Rentals in Brussels under 1,500 euro (Immovlan)**: Scrape rental listings in Brussels up to 1,500 euro per month from Immovlan.be with price, location, EPC label and status. Schedule it to catch new rentals daily.

```json
{
  "location": "Brussels",
  "listing_type": "rent",
  "max_price": 1500,
  "limit": 50,
  "includeDetails": false,
  "trackHistory": false
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `title`, `listing_type`, `price`, `city`, `property_type`, `living_area_m2`, `bedrooms`, `status`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `<source>:<site id>` |
| `source` | string | Short site name |
| `url` | string | Public listing URL on the site |
| `title` | string | Listing title |
| `listing_type` | string | sale or rent |
| `property_type` | string | Normalized type: house, apartment, land, commercial, parking, room or other |
| `status` | string | available, under_offer, sold or rented |
| `price` | number | Asking price in EUR, or monthly rent. For a new-build price range this is the lowest price (null when the price is on request) |
| `price_currency` | string | ISO currency code, EUR |
| `price_period` | string | month for rentals, null for sales |
| `price_type` | string | asking, range or on_request |
| `address` | string | Full address line |
| `street` | string | Street name |
| `house_number` | string | House number with addition |
| `postcode` | string | Postal code |
| `city` | string | City or municipality |
| `region` | string | Region or province |
| `country` | string | ISO country code |
| `lat` | number | Latitude |
| `lon` | number | Longitude |
| `living_area_m2` | number | Living area in m2. Only with the Full details add-on |
| `plot_area_m2` | number | Plot area in m2 |
| `rooms` | integer | Number of rooms |
| `bedrooms` | integer | Number of bedrooms |
| `bathrooms` | integer | Number of bathrooms |
| `energy_label` | string | Energy label / EPC letter |
| `construction_year` | integer | Year built |
| `listed_at` | string | When the listing was first published, ISO 8601 |
| `agent_name` | string | Estate agent or agency |
| `agent_phone` | string | Agent phone number |
| `agent_url` | string | Agent page on the site |
| `image_urls` | array | Photo URLs |
| `description` | string | Listing description in plain text. Only filled with the Full details add-on |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `property_subtype` | string | Immovlan's own type slug, e.g. residence, villa, apartment, ground-floor, office-space, or `project` for a new-build project |
| `price_max` | number | Upper end of a new-build project's price range (null for single listings) |
| `price_updated` | boolean | True when Immovlan shows the 'Price updated' ribbon on the listing |
| `labels` | array | Ribbons shown on the search card: New, Best of, Price updated, Updated, Option, Sold, Rented |
| `snippet` | string | Shortened description from the search card (available without the Full details add-on) |
| `first_seen_at` | string | Add-on Price history: when this listing was first seen by your history store |
| `previous_price` | number | Add-on Price history: the price at the previous run (only for listings seen before) |
| `price_change` | number | Add-on Price history: price now minus previous price |
| `price_change_pct` | number | Add-on Price history: price change in percent |
| `status_changed` | boolean | Add-on Price history: true when the status changed since the previous run |
| `price_history` | array | Add-on Price history: last 20 points `{at, price, status}`; a point is added only when price or status changed |

## Price

Pay per result: $0.50 per 1,000 listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One listing enriched with its detail page: street, coordinates, living and plot area, bedrooms, bathrooms, description, listing date, photos and agent.
- Price history: $1.00 per 1,000. One listing that already had history in your store, annotated with previous price, price change, status change and a price history series. The first sighting of a listing is free.

Store page, reviews and full documentation: [Immovlan Listings Scraper](https://apify.com/carabain-digital/immovlan-listings?fpr=4ijnom)

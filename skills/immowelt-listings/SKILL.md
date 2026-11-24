---
name: immowelt-listings
description: "Scrape real-estate listings from Immowelt (immowelt.de), Germany's big property portal: apartments, houses, land and commercial property for sale and for rent. Filter by city, state or postcode, price, rooms and area. Newest first, with agent phone. Add-on: full details and energy data. Runs the Apify Actor carabain-digital/immowelt-listings. Use for tasks like: Apartments for rent in Hamburg: newest on Immowelt; Houses for sale in Cologne with price history on Immowelt."
---

# Immowelt Scraper - German Property Listings

Scrape real-estate listings from Immowelt (immowelt.de), Germany's big property portal: apartments, houses, land and commercial property for sale and for rent. Filter by city, state or postcode, price, rooms and area. Newest first, with agent phone. Add-on: full details and energy data.

Runs on Apify as the Actor [`carabain-digital/immowelt-listings`](https://apify.com/carabain-digital/immowelt-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 7,692 listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~immowelt-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~immowelt-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"location": "Hamburg", "listing_type": "rent", "property_type": "apartment", "max_price": 1500, "limit": 100, "includeDetails": false}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/immowelt-listings").call(run_input={"location": "Hamburg", "listing_type": "rent", "property_type": "apartment", "max_price": 1500, "limit": 100, "includeDetails": false})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/immowelt-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `location` | string | German city, state, borough or postcode. Several values separated by commas. Example: `Hamburg`, `Berlin, Potsdam` or `Bayern`. Required unless you give search URLs. Ignored when you give search URLs. |
| `listing_type` | string | `sale` for properties to buy, `rent` for rentals. Example: `rent`. Values: `sale`, `rent`. Default: `"sale"`. |
| `property_type` | string | Type of property. `any` = houses and apartments. Example: `apartment`. Values: `any`, `house`, `apartment`, `land`, `commercial`, `parking`. Default: `"any"`. |
| `min_price` | integer | Lowest price in euro (rent: cold rent per month). Example: `200000`, for rent `800`. |
| `max_price` | integer | Highest price in euro (rent: cold rent per month). Example: `450000`, for rent `1500`. |
| `min_rooms` | number | Minimum number of rooms (Zimmer, living room included). Half rooms count: `2.5`. Example: `3`. |
| `min_living_area_m2` | integer | Minimum living area in square metres. Example: `60`. |
| `start_urls` | array | Search result pages copied from immowelt.de; they replace the filters above. Example: `https://www.immowelt.de/classified-search?distributionTypes=Rent&estateTypes=Apartment&locations=AD08DE8634&priceMax=1500`. Filters in the URL are kept; a `/suche/hamburg/wohnungen/mieten` style page works too. Leave empty to use the filters above. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `100`. Immowelt shows at most 10,000 results per search (333 pages): use a city, borough or price range to go deeper. Default: `100`. |
| `includeDetails` | boolean | Also open each listing's page: full description, coordinates (when the exact address is public), construction year, condition, heating, energy certificate data, amenities, deposit and all photos. Adds $1.00 per 1,000 listings on top of the base price. One extra request per listing, so runs take a little longer. Default: `false`. |
| `trackHistory` | boolean | Keep the price and status of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our real-estate Actors use `realestate-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `gent-apartments`. Default: `"realestate-history"`. |

## Examples

**Apartments for rent in Hamburg: newest on Immowelt**: Scrape the newest apartments for rent in Hamburg from Immowelt (immowelt.de): cold rent and service charges, district, rooms, living area, energy label and agent phone. Export to JSON, CSV or Excel.

```json
{
  "location": "Hamburg",
  "listing_type": "rent",
  "property_type": "apartment",
  "max_price": 1500,
  "limit": 100,
  "includeDetails": false
}
```

**Houses for sale in Cologne with price history on Immowelt**: Scrape houses for sale in Cologne (Köln) from Immowelt (immowelt.de) and track their price over time: asking price, price changes, district, living and plot area, energy label and agent. Run it daily to catch price drops.

```json
{
  "location": "Köln",
  "listing_type": "sale",
  "property_type": "house",
  "limit": 200,
  "includeDetails": false,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `updated_at`, `title`, `city`, `price`, `living_area_m2`, `rooms`, `status`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `<source>:<site id>` |
| `source` | string | Short site name |
| `url` | string | Public listing URL on the site |
| `title` | string | Listing title as the agent wrote it (not every listing has one) |
| `listing_type` | string | sale or rent |
| `property_type` | string | house, apartment, land, commercial, parking, room or other |
| `property_subtype` | string | The site's own subtype, e.g. penthouse, loft, single-family-house, villa |
| `status` | string | available, under_offer, sold or rented |
| `price` | number | Asking price in EUR (sale) or rent per month (rent). Null when the price is on request |
| `price_currency` | string | ISO currency code (EUR) |
| `price_period` | string | month for rentals, null for sales |
| `price_type` | string | asking, on_request (Preis auf Anfrage) or auction |
| `address` | string | Street, number, postcode and city on one line. Many sellers hide the street: then only postcode and city |
| `street` | string | Street name. Null when the seller does not publish the exact address |
| `house_number` | string | House number. Null when the seller does not publish the exact address |
| `postcode` | string | German postcode (5 digits) |
| `city` | string | City / municipality |
| `district` | string | City district / borough (Stadtteil), e.g. Winterhude |
| `region` | string | Federal state in English as the site reports it, e.g. Bavaria, North Rhine-Westphalia, Hamburg |
| `country` | string | ISO country code (DE) |
| `lat` | number | Latitude. Add-on Full details only, and only when the exact address is public |
| `lon` | number | Longitude. Add-on Full details only, and only when the exact address is public |
| `living_area_m2` | number | Living area in m2 |
| `plot_area_m2` | number | Plot area in m2 |
| `rooms` | number | Total number of rooms (Zimmer), half rooms as .5 |
| `bedrooms` | integer | Number of bedrooms, when the agent gave it |
| `bathrooms` | integer | Not published as a number by Immowelt: always null |
| `energy_label` | string | Energy efficiency class (A+ to H). Null when the listing has no energy certificate yet |
| `construction_year` | integer | Year of construction. Add-on Full details only |
| `condition` | string | Add-on Full details: condition and building type as the agent wrote it, e.g. neuwertig, Altbau, renoviert / saniert |
| `heating` | string | Add-on Full details: heating system, e.g. Fußbodenheizung |
| `energy_source` | string | Add-on Full details: main energy source, e.g. Gas, Fernwärme |
| `energy_consumption_kwh_m2` | number | Add-on Full details: energy demand or consumption from the energy certificate in kWh per m2 and year |
| `available_from` | string | Move-in date as ISO date, or `immediately`, when the agent gave it |
| `deposit` | string | Add-on Full details, rentals: deposit (Kaution) as the agent wrote it, e.g. 3 Nettokaltmieten |
| `features` | array | Add-on Full details: amenities as listed on the page (kitchen, balcony, cellar, parking, ...) |
| `listed_at` | string | When the listing was first published on Immowelt, ISO 8601 |
| `updated_at` | string | When the listing was last changed on the site, ISO 8601 |
| `agent_name` | string | Estate agent or developer. Null for private sellers |
| `agent_phone` | string | Agent phone number as shown on the listing |
| `agent_url` | string | Agent website |
| `image_urls` | array | Photo URLs |
| `description` | string | Listing description. The search results carry the first ~500 characters (cut off with ...); the add-on Full details gives the complete text |
| `monthly_costs` | number | Rentals: service and heating charges on top of the cold rent (warm rent minus cold rent), when the agent gave both |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: true when the status changed since the previous run |
| `price_history` | array | Price history add-on: price and status points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $0.65 per 1,000 listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One listing enriched with its exposé page: full description, coordinates (when the exact address is public), construction year, condition, heating, energy certificate values, amenities, deposit and all photos.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [Immowelt Scraper - German Property Listings](https://apify.com/carabain-digital/immowelt-listings?fpr=4ijnom)

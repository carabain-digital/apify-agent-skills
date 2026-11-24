---
name: immoscout24-listings
description: "Scrape property listings from ImmoScout24 (immobilienscout24.de), Germany's biggest real-estate portal: apartments, houses, land, garages and WG rooms for rent or sale. Filter by city, price, rooms and area. Newest first. Add-ons: full details with agent phone, price history. Runs the Apify Actor carabain-digital/immoscout24-listings. Use for tasks like: Apartments for rent in Berlin: newest on ImmoScout24; Houses for sale in Munich with price history on ImmoScout24."
---

# ImmoScout24 Scraper - German Property Listings

Scrape property listings from ImmoScout24 (immobilienscout24.de), Germany's biggest real-estate portal: apartments, houses, land, garages and WG rooms for rent or sale. Filter by city, price, rooms and area. Newest first. Add-ons: full details with agent phone, price history.

Runs on Apify as the Actor [`carabain-digital/immoscout24-listings`](https://apify.com/carabain-digital/immoscout24-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 7,692 listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~immoscout24-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~immoscout24-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"location": "Berlin", "listing_type": "rent", "property_type": "apartment", "max_price": 1500, "limit": 100, "includeDetails": true}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/immoscout24-listings").call(run_input={"location": "Berlin", "listing_type": "rent", "property_type": "apartment", "max_price": 1500, "limit": 100, "includeDetails": true})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/immoscout24-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `location` | string | German city, district or state as ImmoScout24's location box knows it. Several separated by commas. Example: `Berlin`, `München`, `Köln - Ehrenfeld` or `Bayern`. Postcodes are not supported (use the district name). Leave empty for all of Germany. Ignored when you give search URLs. |
| `listing_type` | string | `sale` for properties to buy, `rent` for rentals. Example: `rent`. Values: `sale`, `rent`. Default: `"sale"`. |
| `property_type` | string | Type of property. `any` = apartments and houses. Example: `apartment`. Values: `any`, `apartment`, `house`, `land`, `parking`, `room`. Default: `"any"`. |
| `min_price` | integer | Lowest price in euro (rent: cold rent per month). Example: `200000`, for rent `800`. |
| `max_price` | integer | Highest price in euro (rent: cold rent per month). Example: `450000`, for rent `1500`. |
| `min_rooms` | integer | Minimum number of rooms (German count: living rooms and bedrooms, kitchen and bathroom excluded). Example: `2`. |
| `min_living_area_m2` | integer | Minimum living area in square metres. Example: `60`. |
| `start_urls` | array | Search result pages copied from immobilienscout24.de; they replace the filters above. Example: `https://www.immobilienscout24.de/Suche/de/berlin/berlin/wohnung-mieten?price=-1500.0`. Filters in the URL (price, rooms, living space ...) are kept; results always come newest first. Region and radius searches are supported, drawn map areas are not. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `100`. Default: `100`. |
| `includeDetails` | boolean | Also open each listing's exposé: description, features and location texts, all attributes (floor, deposit, heating, energy certificate ...), bedrooms, bathrooms, plot area, construction year, total rent and service charges, state, agent name, phone and profile, all photos. Adds $1.00 per 1,000 listings on top of the base price. One extra request per listing, so runs take a little longer (about 6 s per 100 listings). Default: `false`. |
| `trackHistory` | boolean | Keep the price and status of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our real-estate Actors use `realestate-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `gent-apartments`. Default: `"realestate-history"`. |

## Examples

**Apartments for rent in Berlin: newest on ImmoScout24**: Scrape the newest apartments for rent in Berlin from ImmoScout24 (immobilienscout24.de): cold and total rent, service charges, address, district, rooms, living area, energy class and agent. Export to JSON, CSV or Excel.

```json
{
  "location": "Berlin",
  "listing_type": "rent",
  "property_type": "apartment",
  "max_price": 1500,
  "limit": 100,
  "includeDetails": true
}
```

**Houses for sale in Munich with price history on ImmoScout24**: Scrape houses for sale in Munich from ImmoScout24 (immobilienscout24.de) and track their price over time: asking price, price changes, district, living and plot area, energy class. Run it daily to catch price drops.

```json
{
  "location": "München",
  "listing_type": "sale",
  "property_type": "house",
  "limit": 200,
  "includeDetails": false,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `title`, `city`, `district`, `price`, `living_area_m2`, `rooms`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `<source>:<site id>` |
| `source` | string | Short site name |
| `url` | string | Public exposé URL on immobilienscout24.de |
| `title` | string | Listing title as the provider wrote it |
| `listing_type` | string | sale or rent |
| `property_type` | string | house, apartment, land, commercial, parking, room or other |
| `property_subtype` | string | Details add-on: building or flat type, e.g. villa, multi-family-house, apartment, roof-storey |
| `status` | string | available, under_offer, sold or rented |
| `price` | number | Asking price in EUR (sale) or cold rent (Kaltmiete) per month (rent). Null when the price is on request |
| `price_currency` | string | ISO currency code (EUR) |
| `price_period` | string | month for rentals, null for sales |
| `price_type` | string | asking or on_request |
| `address` | string | Street, number, postcode and city on one line |
| `street` | string | Street name |
| `house_number` | string | House number with addition. Often hidden until you contact the provider |
| `postcode` | string | German postcode (5 digits) |
| `city` | string | City / municipality |
| `district` | string | District / quarter as the site shows it, e.g. Neukölln |
| `region` | string | Details add-on: federal state (Bundesland), e.g. Bayern |
| `country` | string | ISO country code (DE) |
| `lat` | number | Latitude (only when the full address is public) |
| `lon` | number | Longitude (only when the full address is public) |
| `living_area_m2` | number | Living area in m2 |
| `plot_area_m2` | number | Plot area in m2 |
| `rooms` | number | Number of rooms (German count: living rooms and bedrooms, can be 2.5) |
| `bedrooms` | integer | Number of bedrooms |
| `bathrooms` | integer | Number of bathrooms |
| `energy_label` | string | Energy efficiency class from the energy certificate (A+ to H) |
| `construction_year` | integer | Year of construction |
| `listed_at` | string | When the listing was published, ISO 8601. Approximate (from the site's 'vor 3 Stunden'); null when older than a few weeks |
| `private_offer` | boolean | True for private landlords and sellers |
| `agent_name` | string | Details add-on: agency or provider name |
| `agent_contact_name` | string | Details add-on: contact person |
| `agent_phone` | string | Agent phone number |
| `agent_url` | string | Details add-on: provider profile on ImmoScout24 |
| `image_urls` | array | Photo URLs |
| `description` | string | Details add-on: the description text (Objektbeschreibung), German as the provider wrote it |
| `features_text` | string | Details add-on: features text (Ausstattung) |
| `location_text` | string | Details add-on: location text (Lage) |
| `other_text` | string | Details add-on: other information (Sonstiges) |
| `attributes` | object | Details add-on: every attribute of the exposé as German label -> value (floor, deposit, heating, energy demand, balcony, lift ...); checkbox features are true |
| `monthly_costs` | number | Details add-on, rentals: service charges (Nebenkosten) per month |
| `total_rent` | number | Details add-on, rentals: total rent (Gesamtmiete / Warmmiete) per month |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: true when the status changed since the previous run |
| `price_history` | array | Price history add-on: price and status points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $0.65 per 1,000 listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One listing enriched with its exposé: description, features and location texts, all attributes, bedrooms, bathrooms, plot area, construction year, total rent and service charges, federal state, agent name, phone and profile, all photos.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [ImmoScout24 Scraper - German Property Listings](https://apify.com/carabain-digital/immoscout24-listings?fpr=4ijnom)

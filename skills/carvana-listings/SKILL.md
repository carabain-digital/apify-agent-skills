---
name: carvana-listings
description: "Scrape used car listings from Carvana, the biggest US online car dealer: price, mileage, year, VIN, KBB value, price drops, body, fuel, colour and photos. Filter by make, model, year, price, mileage, fuel and body type. Add-ons: full details, price history. Runs the Apify Actor carabain-digital/carvana-listings. Use for tasks like: Used Toyota RAV4 under $30,000 on Carvana; Used electric cars on Carvana with price history."
---

# Carvana Car Listings Scraper

Scrape used car listings from Carvana, the biggest US online car dealer: price, mileage, year, VIN, KBB value, price drops, body, fuel, colour and photos. Filter by make, model, year, price, mileage, fuel and body type. Add-ons: full details, price history.

Runs on Apify as the Actor [`carabain-digital/carvana-listings`](https://apify.com/carabain-digital/carvana-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 5,000 car listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~carvana-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~carvana-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"make": "Toyota", "model": "RAV4", "condition": "used", "max_price": 30000, "limit": 200}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/carvana-listings").call(run_input={"make": "Toyota", "model": "RAV4", "condition": "used", "max_price": 30000, "limit": 200})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/carvana-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `make` | string | Car make, any capitalisation. Example: `Toyota`, `Ford`, `Mercedes-Benz`, `Land Rover`. Leave empty for all makes. An unknown make stops the run with an error instead of returning nothing. |
| `model` | string | Model of that make as Carvana writes it. Example: `RAV4`, `F-150`, `Civic`, `Model 3`. Needs a make. If a model is not found, run the search on carvana.com and paste its URL into Search URLs. |
| `min_year` | integer | Earliest model year. Example: `2018`. |
| `max_year` | integer | Latest model year. Example: `2022`. |
| `min_price` | integer | Lowest asking price in US dollars. Example: `10000`. |
| `max_price` | integer | Highest asking price in US dollars. Example: `30000`. |
| `max_mileage` | integer | Highest odometer reading in miles. Example: `60000`. |
| `fuel_type` | string | Fuel type. Example: `electric`. Values: `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`. |
| `transmission` | string | Gearbox. Example: `automatic`. Values: `automatic`, `manual`. |
| `body_type` | string | Body type. Example: `pickup`. Values: `sedan`, `hatchback`, `wagon`, `suv`, `coupe`, `convertible`, `minivan`, `pickup`. |
| `condition` | string | Carvana mostly sells used cars. Example: `used`. `new` returns the few thousand new cars Carvana also lists. Values: `used`, `any`, `new`. Default: `"used"`. |
| `start_urls` | array | Search result pages copied from carvana.com; they replace the filters above. Example: `https://www.carvana.com/cars/honda-civic` or any filtered results URL. Results of URLs with a `cvnaid` come newest first; plain `/cars/<make>-<model>` URLs keep the site's popularity order. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many cars (newest first). Example: `500`. Carvana shows about 21 cars per page, so 500 cars take about 24 pages. Default: `100`. |
| `includeDetails` | boolean | Also open each car's page: engine, transmission, drivetrain, doors, MPG, Carvana facility address, features by category, inspection summary, recalls, prior use (rental, lease), accident flag, warranty and dimensions. Costs extra per enriched car and is slower (about 1-2 s per car). Default: `false`. |
| `trackHistory` | boolean | Keep the price of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct and price_history. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Example: `golf-watch`. Default: `"cars-history"`. |

## Examples

**Used Toyota RAV4 under $30,000 on Carvana**: Scrape the newest used Toyota RAV4 listings under $30,000 from Carvana: price, KBB value, mileage, year, VIN, price drops and photos. Export to JSON, CSV or Excel.

```json
{
  "make": "Toyota",
  "model": "RAV4",
  "condition": "used",
  "max_price": 30000,
  "limit": 200
}
```

**Used electric cars on Carvana with price history**: Track used electric cars on Carvana over time: price, price drops, KBB value, mileage, year and VIN. Run it daily to catch price cuts.

```json
{
  "fuel_type": "electric",
  "condition": "used",
  "limit": 500,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `title`, `year`, `mileage`, `fuel_type`, `price`, `price_vs_kbb`, `badges`, `vin`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `carvana:<Carvana vehicle id>` |
| `source` | string | Short site name: carvana |
| `url` | string | Public car page on carvana.com |
| `title` | string | Year, make, model and trim, e.g. 2024 GMC Terrain SLE |
| `make` | string | Make, e.g. BMW |
| `model` | string | Model, e.g. X1 |
| `variant` | string | Trim as Carvana writes it, e.g. SLE |
| `condition` | string | used or new |
| `price` | integer | Carvana's price in US dollars (no fees to add on top: taxes and registration are charged at checkout) |
| `price_currency` | string | USD |
| `price_type` | string | fixed or on_request |
| `mileage` | integer | Odometer in miles |
| `mileage_unit` | string | Always mi |
| `year` | integer | Model year |
| `first_registration` | string | Not published by Carvana (null) |
| `fuel_type` | string | petrol, diesel, electric, hybrid, plugin_hybrid, ethanol or hydrogen |
| `transmission` | string | Details add-on: automatic or manual |
| `body_type` | string | sedan, hatchback, wagon, suv, coupe, convertible, van, pickup, minivan or other |
| `power_kw` | integer | Power in kW (from the horsepower) |
| `power_hp` | integer | Horsepower |
| `engine_cc` | integer | Details add-on: engine displacement in cc (from the litres) |
| `color` | string | Exterior colour (search: generic colour; details add-on: the maker's colour name) |
| `doors` | integer | Details add-on: number of doors |
| `seats` | integer | Number of seats |
| `vin` | string | Vehicle identification number |
| `seller_type` | string | Always dealer |
| `seller_name` | string | Always Carvana |
| `seller_phone` | string | Not published (null) |
| `seller_url` | string | https://www.carvana.com |
| `address` | string | Details add-on: address of the Carvana facility where the car is stored |
| `city` | string | Details add-on: city of that facility |
| `postcode` | string | Details add-on: ZIP code of that facility |
| `region` | string | Details add-on: US state code of that facility |
| `country` | string | Always US |
| `lat` | number | Not published (null) |
| `lon` | number | Not published (null) |
| `listed_at` | string | Not published by Carvana (null); use `price_updated_at` |
| `image_urls` | array | Photo URLs (main photo and the hero image) |
| `description` | string | Details add-on: Carvana's highlights of the car |
| `badges` | array | Tags on the card, e.g. New Price, Great Deal, Recently Added |
| `msrp` | number | Manufacturer's suggested retail price when the car was new |
| `drivetrain` | string | Details add-on: drivetrain, e.g. AWD |
| `interior_color` | string | Interior colour |
| `details_fetched` | boolean | True when the Full details add-on enriched this item |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price before Carvana's latest price change on the site; with the Price history add-on: the price at your previous run |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: true when the status changed |
| `stock_number` | string | Carvana stock number |
| `kbb_value` | number | Kelley Blue Book value Carvana shows, USD |
| `price_vs_kbb` | number | Price minus KBB value (negative = below KBB) |
| `price_updated_at` | string | When the price last changed on carvana.com (ISO 8601) |
| `mpg` | number | Combined MPG from the search card |
| `trim_kbb` | string | KBB trim name, e.g. SLE Sport Utility 4D |
| `purchase_pending` | boolean | True when someone has this car in checkout |
| `facility_id` | number | Carvana facility id the car is stored at |
| `engine` | string | Details add-on: engine, e.g. 1.5L I-4 DOHC |
| `transmission_text` | string | Details add-on: transmission as Carvana writes it |
| `mpg_city` | number | Details add-on: city MPG |
| `mpg_highway` | number | Details add-on: highway MPG |
| `mpg_combined` | number | Details add-on: combined MPG |
| `in_service_date` | string | Details add-on: date the car first went into service (YYYY-MM-DD) |
| `accident_reported` | boolean | Details add-on: Autocheck reports an accident |
| `prior_use` | array | Details add-on: previous use, e.g. Rental, Lease |
| `number_of_keys` | number | Details add-on: number of keys |
| `recalls` | array | Details add-on: open recalls |
| `equipment` | object | Details add-on: features by category (Safety & Security, Comfort & Convenience, ...) |
| `installed_options` | array | Details add-on: factory options and packages installed |
| `inspection` | object | Details add-on: Carvana's inspection summary `{inspected_on, points, passed, fixed_or_replaced}` |
| `warranty` | object | Details add-on: remaining manufacturer warranty in months and miles |
| `dimensions` | object | Details add-on: length, width, height, wheelbase, curb weight, towing and trunk capacity |
| `price_history` | array | Price history add-on: price points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $1.00 per 1,000 car listings on the Apify Free plan, less on paid plans, plus $0.025 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- Full details: $3.00 per 1,000. One car listing enriched with its Carvana page: engine, transmission, drivetrain, doors, MPG, Carvana facility address, features by category, inspection summary, recalls, prior use (rental/lease), accident flag, warranty and dimensions.
- Price history: $1.00 per 1,000. One car listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [Carvana Car Listings Scraper](https://apify.com/carabain-digital/carvana-listings?fpr=4ijnom)

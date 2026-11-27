---
name: carmax-listings
description: "Scrape used car listings from CarMax (carmax.com), the biggest US used-car retailer: price, mileage, VIN, specs, features, store, rating and shipping fee. Search nationwide or by ZIP; filter by make, model, year, price, mileage, fuel, body. Newest first. Add-ons: full details, price history. Runs the Apify Actor carabain-digital/carmax-listings. Use for tasks like: Newest used Honda Civic at CarMax, all US stores; Used hybrid SUVs near Miami at CarMax: price history."
---

# CarMax Car Listings Scraper

Scrape used car listings from CarMax (carmax.com), the biggest US used-car retailer: price, mileage, VIN, specs, features, store, rating and shipping fee. Search nationwide or by ZIP; filter by make, model, year, price, mileage, fuel, body. Newest first. Add-ons: full details, price history.

Runs on Apify as the Actor [`carabain-digital/carmax-listings`](https://apify.com/carabain-digital/carmax-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 5,000 car listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~carmax-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~carmax-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"make": "Honda", "model": "Civic", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/carmax-listings").call(run_input={"make": "Honda", "model": "Civic", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/carmax-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `location` | string | US ZIP code to search around (5 digits). Example: `90210` (Beverly Hills), `10001`, `60601`. Leave empty to search all CarMax stores in the US. Ignored for the distance when you give search URLs that carry their own `zip`. |
| `radius_km` | integer | Distance from the ZIP code in kilometres. CarMax offers 5, 25, 50, 75, 100, 250 and 500 miles: your value is rounded up to the next one (default 50 miles; above 500 miles = nationwide). Cars at far stores that CarMax ships are included only inside the radius. Example: `160` for 100 miles. |
| `make` | string | Car make as CarMax writes it. Example: `Toyota`, `Ford`, `Mercedes-Benz`, `Land Rover`. CarMax sells 36 makes (Acura to Volvo). |
| `model` | string | Model of that make (needs Make), as the site writes it. Example: `Camry`, `F-150`, `Model 3`, `CR-V`. An unknown model fails with an error. |
| `min_year` | integer | Oldest model year. Example: `2018`. |
| `max_year` | integer | Newest model year. Example: `2023`. |
| `min_price` | integer | Lowest price in US dollars. Example: `10000`. |
| `max_price` | integer | Highest price in US dollars. Example: `25000`. |
| `max_mileage` | integer | Highest odometer reading in miles. Example: `60000`. |
| `fuel_type` | string | Fuel type. Example: `electric`. Values: `any`, `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`. Default: `"any"`. |
| `transmission` | string | Transmission. Example: `manual`. Values: `any`, `automatic`, `manual`. Default: `"any"`. |
| `body_type` | string | Body style. Example: `suv`. Values: `any`, `sedan`, `suv`, `pickup`, `hatchback`, `coupe`, `convertible`, `wagon`, `minivan`, `van`. Default: `"any"`. |
| `start_urls` | array | Search result pages copied from carmax.com; they replace the filters above. Example: `https://www.carmax.com/cars/honda/civic` or `https://www.carmax.com/cars/suvs/hybrid?year=2021-2024&price=-35000&zip=33101&distance=50`. Filters in the URL (make, model, body, fuel, transmission, year, price, mileage, ZIP and distance) are kept; results come newest first unless the URL sets its own `sort`. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `100`. Default: `100`. |
| `includeDetails` | boolean | Also open each car page: accident and owner count from the vehicle history, store coordinates, prior use (rental, personal ...), the full feature list, warranty and model generation. Adds $1.00 per 1,000 listings on top of the base price. One extra request per listing (about 1 s per listing). Default: `false`. |
| `trackHistory` | boolean | Keep the price of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `camry-nyc`. Default: `"cars-history"`. |

## Examples

**Newest used Honda Civic at CarMax, all US stores**: Scrape the newest used Honda Civic listings from every CarMax store in the US: price, mileage, VIN, year, trim, features, store and shipping fee. Export to JSON, CSV or Excel.

```json
{
  "make": "Honda",
  "model": "Civic",
  "limit": 100
}
```

**Used hybrid SUVs near Miami at CarMax: price history**: Scrape used hybrid SUVs from 2021 on under $35,000 within 50 miles of Miami (ZIP 33101) from CarMax and track their price over time: price drops, mileage, VIN and store. Run it daily to catch price drops.

```json
{
  "location": "33101",
  "radius_km": 80,
  "body_type": "suv",
  "fuel_type": "hybrid",
  "min_year": 2021,
  "max_price": 35000,
  "limit": 200,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `title`, `price`, `mileage`, `city`, `region`, `seller_name`, `vin`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `carmax:<stock number>` |
| `source` | string | Short site name (carmax) |
| `url` | string | Public car page on carmax.com |
| `title` | string | Year, make, model and trim, e.g. 2017 Honda Civic EX |
| `make` | string | Make, e.g. Toyota |
| `model` | string | Model, e.g. Camry |
| `variant` | string | Trim, e.g. EX |
| `condition` | string | Always used: CarMax sells used cars only |
| `price` | number | CarMax's no-haggle price in USD (excludes tax, title and registration) |
| `price_currency` | string | USD |
| `price_type` | string | fixed |
| `mileage` | integer | Odometer reading |
| `mileage_unit` | string | mi (miles) |
| `year` | integer | Model year |
| `first_registration` | string | Always null: US listings only have the model year |
| `fuel_type` | string | petrol, diesel, electric, hybrid or plugin_hybrid |
| `transmission` | string | manual or automatic |
| `body_type` | string | sedan, hatchback, wagon, suv, coupe, convertible, van, pickup or minivan (CarMax's body type group) |
| `power_kw` | integer | Engine power in kW (from hp) |
| `power_hp` | integer | Engine power in hp, when CarMax knows it |
| `engine_cc` | integer | Engine displacement in cc |
| `color` | string | Exterior colour |
| `doors` | integer | Number of doors (from the body style, e.g. 4D Sedan) |
| `seats` | integer | Always null: not published |
| `vin` | string | Vehicle identification number (17 characters) |
| `seller_type` | string | dealer (CarMax stores) |
| `seller_name` | string | CarMax store, e.g. CarMax Murrieta |
| `seller_phone` | string | Always null: not published on the listing |
| `seller_url` | string | Always null |
| `address` | string | Store city and state (the street address is not published) |
| `city` | string | Store city |
| `postcode` | string | Always null: not published |
| `region` | string | Store state, e.g. CA |
| `country` | string | US |
| `lat` | number | Details add-on: store latitude |
| `lon` | number | Details add-on: store longitude |
| `listed_at` | string | Date the car went on sale at CarMax (YYYY-MM-DD) |
| `image_urls` | array | Hero photo URL (add ?width=800 for a size) |
| `description` | string | Always null: CarMax shows no free-text description |
| `original_price` | number | Price before the latest price drop, when the car had one |
| `price_drop` | boolean | True when the price was recently lowered |
| `new_arrival` | boolean | True for a car that just arrived on the lot |
| `body_style` | string | Body style as CarMax shows it, e.g. 4D Sedan |
| `highlights` | array | CarMax highlight tags, e.g. singleOwner, lowMilesPerYear, fuelEfficient |
| `store_id` | integer | CarMax store number |
| `shipping_fee` | number | Shipping fee in USD when the car is at a far store (0 = free); null when it is at the searched store |
| `shipping_text` | string | Shipping/test-drive text, e.g. $349 Shipping |
| `reserved` | boolean | True when the car is reserved by another customer |
| `prior_use` | array | Details add-on: earlier uses from the history, e.g. Leased Vehicle, Rental |
| `warranty` | array | Details add-on: warranties `[{type, years, miles}]` |
| `generation` | string | Details add-on: model generation, e.g. Ninth generation |
| `drivetrain` | string | e.g. Front Wheel Drive |
| `engine` | string | e.g. 4-cyl, Gas, 1.8L |
| `fuel_type_text` | string | Engine type as CarMax shows it: Gas, Diesel, Electric, Hybrid, Plug-In Hybrid |
| `interior_color` | string | Interior colour |
| `mpg_city` | number | City fuel economy (MPG) |
| `mpg_highway` | number | Highway fuel economy (MPG) |
| `options` | array | Features from the search card, e.g. Bluetooth Technology |
| `stock_number` | string | CarMax stock number (the number in the car page URL) |
| `distance_mi` | number | Distance from the searched ZIP in miles (without a ZIP: from the proxy exit, not meaningful) |
| `seller_rating` | number | CarMax store rating (0-5) |
| `seller_review_count` | integer | Number of reviews of the store |
| `equipment` | object | Details add-on: `{Features: [...], "Standard equipment": [...]}` |
| `accident_count` | integer | Details add-on: accidents in the vehicle history report |
| `owner_count` | integer | Details add-on: previous owners in the vehicle history report |
| `details_fetched` | boolean | True when the details add-on enriched this item |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: always false for CarMax (no status field) |
| `price_history` | array | Price history add-on: price points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $1.00 per 1,000 car listings on the Apify Free plan, less on paid plans, plus $0.025 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One listing enriched with its car page: accident and owner count, store coordinates, prior use (rental, personal), full feature list, warranty and model generation.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [CarMax Car Listings Scraper](https://apify.com/carabain-digital/carmax-listings?fpr=4ijnom)

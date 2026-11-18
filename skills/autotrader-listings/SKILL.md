---
name: autotrader-listings
description: "Scrape car listings from Autotrader.com (USA): new, used and certified cars from dealers and private sellers by ZIP code and radius. Filter by make, model, year, price, mileage, fuel, body. Price, VIN, KBB fair price, deal rating, dealer phone. Newest first. Runs the Apify Actor carabain-digital/autotrader-listings. Use for tasks like: Used Toyota Camry 2018-2023 near New York on Autotrader; Electric cars near Los Angeles with price history on Autotrader."
---

# Autotrader Car Listings Scraper

Scrape car listings from Autotrader.com (USA): new, used and certified cars from dealers and private sellers by ZIP code and radius. Filter by make, model, year, price, mileage, fuel, body. Price, VIN, KBB fair price, deal rating, dealer phone. Newest first.

Runs on Apify as the Actor [`carabain-digital/autotrader-listings`](https://apify.com/carabain-digital/autotrader-listings). The user needs an Apify account and an API token ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)); runs are billed to their account.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~autotrader-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~autotrader-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"make": "Toyota", "model": "Camry", "condition": "used", "min_year": 2018, "max_year": 2023, "min_price": 10000, "max_price": 30000, "max_mileage": 80000, "location": "10001", "radius_km": 160, "limit": 200}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/autotrader-listings").call(run_input={"make": "Toyota", "model": "Camry", "condition": "used", "min_year": 2018, "max_year": 2023, "min_price": 10000, "max_price": 30000, "max_mileage": 80000, "location": "10001", "radius_km": 160, "limit": 200})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/autotrader-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `make` | string | Car make as Autotrader names it. Example: `Toyota`, `Mercedes-Benz`, `Land Rover`. Leave empty for all makes. An unknown make fails the run instead of returning every car. |
| `model` | string | Model of that make. Example: `Camry`, `CR-V`, `F-150`, `3 Series`. Needs a make. |
| `min_year` | integer | Oldest model year. Example: `2018`. |
| `max_year` | integer | Newest model year. Example: `2023`. |
| `min_price` | integer | Lowest asking price in US dollars. Example: `10000`. |
| `max_price` | integer | Highest asking price in US dollars. Example: `30000`. |
| `max_mileage` | integer | Highest odometer reading in miles. Example: `60000`. |
| `fuel_type` | string | Only cars with this fuel. Example: `electric`. Values: `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`, `hydrogen`. |
| `transmission` | string | Only automatic or only manual cars. Example: `manual`. Values: `automatic`, `manual`. |
| `body_type` | string | Only this body style. Example: `suv`. Van and minivan are one category on Autotrader. Values: `sedan`, `hatchback`, `wagon`, `suv`, `coupe`, `convertible`, `van`, `minivan`, `pickup`. |
| `condition` | string | New, used (certified pre-owned included) or both. Example: `used`. Values: `any`, `used`, `new`. Default: `"any"`. |
| `seller_type` | string | Who sells the car. Example: `private`. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `location` | string | US ZIP code to search around (5 digits). Example: `10001` (New York), `90012` (Los Angeles), `60601` (Chicago). Leave empty to search nationwide. |
| `radius_km` | integer | Search radius around the ZIP code in kilometres; it is rounded up to Autotrader's options (10, 25, 50, 75, 100, 200, 300, 400, 500 miles; beyond 500 miles = nationwide). Example: `80` (= 50 miles), `160` (= 100 miles). Default: `80`. |
| `start_urls` | array | Search result pages copied from autotrader.com; they replace the filters above and keep every filter and the sort order in the URL. Example: `https://www.autotrader.com/cars-for-sale/used-cars/honda/cr-v/los-angeles-ca?zip=90012&searchRadius=50&startYear=2020`. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `100`. Default: `100`. |
| `includeDetails` | boolean | Also open each listing's page: the dealer's full description, every photo, equipment by category, highlighted features, option packages, original warranties, open safety recalls, walk-around videos, all dealer phone numbers and opening hours. Adds $1.00 per 1,000 listings on top of the base price. One extra request per listing (about 3 s per 10 listings). Default: `false`. |
| `trackHistory` | boolean | Keep the price of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `nyc-used-suvs`. Default: `"cars-history"`. |

## Examples

**Used Toyota Camry 2018-2023 near New York on Autotrader**: Scrape used Toyota Camry listings (2018-2023, $10k-$30k, under 80,000 miles) within 100 miles of New York from Autotrader.com: price, mileage, VIN, KBB fair price, deal rating, dealer and phone. Export to JSON, CSV or Excel.

```json
{
  "make": "Toyota",
  "model": "Camry",
  "condition": "used",
  "min_year": 2018,
  "max_year": 2023,
  "min_price": 10000,
  "max_price": 30000,
  "max_mileage": 80000,
  "location": "10001",
  "radius_km": 160,
  "limit": 200
}
```

**Electric cars near Los Angeles with price history on Autotrader**: Scrape electric cars for sale within 50 miles of Los Angeles from Autotrader.com and track their price over time: asking price, price drops, KBB fair price, mileage, dealer. Run it daily to catch price cuts.

```json
{
  "fuel_type": "electric",
  "location": "90012",
  "radius_km": 80,
  "limit": 300,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `title`, `year`, `mileage`, `price`, `deal_rating`, `city`, `region`, `seller_name`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `autotrader:<listing id>` |
| `source` | string | Short site name (autotrader) |
| `url` | string | Public listing URL on autotrader.com |
| `title` | string | Listing title, e.g. `Used 2025 Buick Encore GX Avenir` |
| `make` | string | Make, e.g. Toyota |
| `model` | string | Model, e.g. Camry |
| `variant` | string | Trim, e.g. XLE (null when the site has none) |
| `condition` | string | new or used (certified pre-owned cars are used, see certified) |
| `price` | number | Asking price in USD (the dealer's sale price). Null when the dealer shows 'Contact Dealer For Price' |
| `price_currency` | string | USD |
| `price_type` | string | fixed, or on_request when there is no price |
| `mileage` | integer | Odometer reading in miles |
| `mileage_unit` | string | mi |
| `year` | integer | Model year |
| `first_registration` | string | Model year as YYYY (Autotrader does not publish a registration date) |
| `fuel_type` | string | petrol, diesel, electric, hybrid, plugin_hybrid, hydrogen, ethanol (flex fuel), cng, lpg or other |
| `transmission` | string | automatic or manual |
| `body_type` | string | sedan, hatchback, wagon, suv, coupe, convertible, van, minivan, pickup or other |
| `power_kw` | number | Not published by Autotrader (null) |
| `power_hp` | number | Not published by Autotrader (null) |
| `engine_cc` | integer | Engine displacement in cc, from the litres the site shows (2.0 L = 2000) |
| `color` | string | Exterior colour as the dealer wrote it |
| `doors` | integer | Number of doors |
| `seats` | integer | Not published in search results (null) |
| `vin` | string | Vehicle identification number (17 characters) |
| `seller_type` | string | dealer or private |
| `seller_name` | string | Dealer name, or 'Private Seller Exchange' for private sellers |
| `seller_phone` | string | Seller phone number in +1 format (the dealer's tracking number for this listing) |
| `seller_url` | string | Dealer website link for this car |
| `address` | string | Dealer street, city, state and ZIP on one line |
| `city` | string | Seller city |
| `postcode` | string | Seller ZIP code |
| `region` | string | US state code, e.g. NY |
| `country` | string | US |
| `lat` | number | Dealer latitude |
| `lon` | number | Dealer longitude |
| `listed_at` | string | Date the car was listed on Autotrader (YYYY-MM-DD, from its days on site) |
| `image_urls` | array | Photo URLs (search results carry the main photo; the details add-on adds every photo) |
| `description` | string | Details add-on: the dealer's full description |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `certified` | boolean | True for manufacturer or third-party certified pre-owned cars |
| `trim` | string | Trim name as the site shows it |
| `drive_type` | string | Drive type, e.g. All wheel drive |
| `engine` | string | Engine, e.g. 4-Cylinder Turbo |
| `engine_displacement_l` | number | Engine displacement in litres |
| `interior_color` | string | Interior colour |
| `mpg_city` | number | City fuel economy in MPG |
| `mpg_highway` | number | Highway fuel economy in MPG |
| `stock_number` | string | Dealer stock number |
| `days_on_site` | integer | Days the car has been listed on Autotrader |
| `deal_rating` | string | Autotrader/KBB deal rating: Great, Good or null |
| `kbb_fair_price` | number | Kelley Blue Book Fair Purchase Price in USD |
| `kbb_fair_price_low` | number | Low end of the KBB fair price range |
| `kbb_fair_price_high` | number | High end of the KBB fair price range |
| `price_reduced` | boolean | True when the dealer recently cut the price |
| `site_price_history` | array | Price changes Autotrader shows for this listing, `[{date, price}]` |
| `vehicle_history` | array | Vehicle history badges, e.g. NO_ACCIDENTS_REPORTED, ONE_OWNER, NO_SALVAGE_TITLE |
| `dealer_rating` | number | Dealer review score (0-5) |
| `dealer_review_count` | integer | Number of dealer reviews |
| `distance_mi` | number | Distance from the searched ZIP code in miles |
| `details_fetched` | boolean | True when the details add-on enriched this item |
| `features` | object | Details add-on: equipment by category (exterior, interior, mechanical, safety, technology, other) |
| `highlights` | array | Details add-on: the key features the site highlights, e.g. Backup Camera, Apple CarPlay |
| `packages` | array | Details add-on: option packages |
| `warranties` | object | Details add-on: original warranties, e.g. {basic: '3 Years/36,000 Miles'} |
| `open_recalls` | integer | Details add-on: number of open safety recalls |
| `video_urls` | array | Details add-on: walk-around video URLs |
| `seller_phones` | array | Details add-on: every phone number of the dealer |
| `dealer_hours` | object | Details add-on: opening hours per weekday |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: true when the status changed since the previous run |
| `price_history` | array | Price history add-on: price and status points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $1.00 per 1,000 car listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One car listing enriched with its Autotrader page: the dealer's full description, every photo, equipment by category, highlighted features, option packages, original warranties, open safety recalls, walk-around videos, all dealer phone numbers and opening hours.
- Price history: $1.00 per 1,000. One car listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [Autotrader Car Listings Scraper](https://apify.com/carabain-digital/autotrader-listings)

---
name: cargurus-listings
description: "Scrape used and new car listings from CarGurus (cargurus.com), the biggest US car marketplace: price, CarGurus deal rating and Instant Market Value, mileage, VIN, specs and dealer phone. Search by ZIP, make, model, year, price, mileage. Newest first. Add-ons: full details, price history. Runs the Apify Actor carabain-digital/cargurus-listings. Use for tasks like: Used Toyota Camry near New York with CarGurus deal ratings; Used electric cars in Los Angeles: CarGurus price history."
---

# CarGurus Car Listings Scraper

Scrape used and new car listings from CarGurus (cargurus.com), the biggest US car marketplace: price, CarGurus deal rating and Instant Market Value, mileage, VIN, specs and dealer phone. Search by ZIP, make, model, year, price, mileage. Newest first. Add-ons: full details, price history.

Runs on Apify as the Actor [`carabain-digital/cargurus-listings`](https://apify.com/carabain-digital/cargurus-listings). The user needs an Apify account and an API token ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)); runs are billed to their account.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~cargurus-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~cargurus-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"location": "10001", "make": "Toyota", "model": "Camry", "condition": "used", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/cargurus-listings").call(run_input={"location": "10001", "make": "Toyota", "model": "Camry", "condition": "used", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/cargurus-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `location` | string | US ZIP code to search around (5 digits), as on CarGurus. Example: `10001` (New York), `90210`, `60601`. Leave empty to search all of the US. Ignored when you give search URLs. |
| `radius_km` | integer | Distance from the ZIP code in kilometres (converted to miles for CarGurus: 80 km = 50 mi). Default 50 miles. Example: `160` for 100 miles. |
| `make` | string | Car make as CarGurus writes it. Example: `Toyota`, `Ford`, `Mercedes-Benz`, `Land Rover`. |
| `model` | string | Model of that make (needs Make). Example: `Camry`, `F-150`, `Model 3`, `CR-V`. |
| `condition` | string | `used` (CarGurus' default), `new`, or `any` (used cars first, then new cars). Example: `used`. Values: `used`, `new`, `any`. Default: `"used"`. |
| `min_year` | integer | Oldest model year. Example: `2018`. |
| `max_year` | integer | Newest model year. Example: `2023`. |
| `min_price` | integer | Lowest price in US dollars. Example: `10000`. |
| `max_price` | integer | Highest price in US dollars (CarGurus filters on the price before dealer fees). Example: `25000`. |
| `max_mileage` | integer | Highest odometer reading in miles. Example: `60000`. |
| `fuel_type` | string | Fuel type. Example: `electric`. Values: `any`, `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`, `ethanol`, `cng`, `hydrogen`. Default: `"any"`. |
| `transmission` | string | Transmission. Example: `manual`. Values: `any`, `automatic`, `manual`. Default: `"any"`. |
| `body_type` | string | Body style. Example: `suv`. Values: `any`, `sedan`, `suv`, `pickup`, `hatchback`, `coupe`, `convertible`, `wagon`, `minivan`, `van`. Default: `"any"`. |
| `seller_type` | string | Keep only dealer or only private-seller listings. CarGurus search results are almost all dealers. Example: `dealer`. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `start_urls` | array | Search result pages copied from cargurus.com; they replace the filters above. Example: `https://www.cargurus.com/Cars/l-Used-Toyota-Camry-d292?zip=10001&distance=50`. Filters in the URL (ZIP, distance, price, year, mileage, body type ...) are kept; results always come newest first. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `100`. Default: `100`. |
| `includeDetails` | boolean | Also open each listing: dealer description, all photos, full equipment list per category, accident and owner count from the vehicle history report, dealer fees and price before fees, dealer street address, coordinates, website and rating. Adds $1.00 per 1,000 listings on top of the base price. One extra request per listing (about 4 s per 10 listings). Default: `false`. |
| `trackHistory` | boolean | Keep the price of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `camry-nyc`. Default: `"cars-history"`. |

## Examples

**Used Toyota Camry near New York with CarGurus deal ratings**: Scrape the newest used Toyota Camry listings within 50 miles of New York (ZIP 10001) from CarGurus: price, deal rating, Instant Market Value, mileage, VIN, dealer and phone. Export to JSON, CSV or Excel.

```json
{
  "location": "10001",
  "make": "Toyota",
  "model": "Camry",
  "condition": "used",
  "limit": 100
}
```

**Used electric cars in Los Angeles: CarGurus price history**: Scrape used electric cars under $40,000 around Los Angeles (ZIP 90012) from CarGurus and track their price over time: price drops, deal rating, market value, mileage and dealer. Run it daily to catch the best EV deals.

```json
{
  "location": "90012",
  "radius_km": 80,
  "fuel_type": "electric",
  "max_price": 40000,
  "limit": 200,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `title`, `price`, `imv_price`, `deal_rating`, `mileage`, `city`, `region`, `seller_name`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `cargurus:<listing id>` |
| `source` | string | Short site name (cargurus) |
| `url` | string | Public listing page on cargurus.com |
| `title` | string | Listing title, e.g. 2019 Honda CR-V LX AWD |
| `make` | string | Make, e.g. Toyota |
| `model` | string | Model, e.g. Camry |
| `variant` | string | Trim, e.g. LX AWD |
| `condition` | string | used or new |
| `price` | number | Listed price in USD (includes dealer fees where the state requires it). Null when the dealer shows no price |
| `price_currency` | string | USD |
| `price_type` | string | fixed, or on_request when there is no price |
| `mileage` | integer | Odometer reading |
| `mileage_unit` | string | mi (miles) |
| `year` | integer | Model year |
| `first_registration` | string | Always null: US listings only have the model year |
| `fuel_type` | string | petrol, diesel, electric, hybrid, plugin_hybrid, ethanol (flex fuel), cng, hydrogen or other |
| `transmission` | string | manual or automatic |
| `body_type` | string | sedan, hatchback, wagon, suv, coupe, convertible, van, pickup, minivan or other |
| `power_kw` | integer | Engine power in kW (from hp) |
| `power_hp` | integer | Engine power in hp, when CarGurus knows it |
| `engine_cc` | integer | Engine displacement in cc |
| `color` | string | Exterior colour as the dealer names it |
| `doors` | integer | Number of doors |
| `seats` | integer | Always null: not published |
| `vin` | string | Vehicle identification number (17 characters) |
| `seller_type` | string | dealer or private |
| `seller_name` | string | Dealer name |
| `seller_phone` | string | Dealer phone number |
| `seller_url` | string | Details add-on: dealer website |
| `address` | string | Dealer address (city and state; full street address with the details add-on) |
| `city` | string | Dealer city |
| `postcode` | string | Dealer ZIP code |
| `region` | string | Dealer state, e.g. NJ |
| `country` | string | US |
| `lat` | number | Details add-on: dealer latitude |
| `lon` | number | Details add-on: dealer longitude |
| `listed_at` | string | Date the car was listed on CarGurus (YYYY-MM-DD, from days on market) |
| `image_urls` | array | Photo URLs (one without details, all with the details add-on) |
| `description` | string | Details add-on: the dealer's description |
| `deal_rating` | string | CarGurus deal rating: great_price, good_price, fair_price, poor_price (shown as High Price), overpriced, na (no rating), outlier; null for new cars |
| `imv_price` | number | CarGurus Instant Market Value (IMV): what similar cars sell for, USD |
| `imv_difference` | number | IMV minus price: positive = below market value, negative = above |
| `deal_score` | number | CarGurus deal score behind the rating |
| `days_on_market` | integer | Days the car has been listed on CarGurus |
| `total_price` | number | CarGurus total price incl. fees where shown |
| `msrp` | number | Original MSRP |
| `certified_pre_owned` | boolean | True for certified pre-owned (CPO) cars |
| `drivetrain` | string | e.g. All-Wheel Drive |
| `engine` | string | Engine as CarGurus shows it, e.g. 184 hp 2.4L I4 |
| `transmission_text` | string | Transmission as CarGurus shows it, e.g. 8-Speed Automatic |
| `fuel_type_text` | string | Fuel type as CarGurus shows it, e.g. Gasoline |
| `interior_color` | string | Interior colour |
| `mpg_city` | number | City fuel economy (MPG) |
| `mpg_highway` | number | Highway fuel economy (MPG) |
| `mpg_combined` | number | Combined fuel economy (MPG) |
| `options` | array | Top options from the search card, e.g. Backup Camera |
| `stock_number` | string | Dealer stock number |
| `distance_mi` | number | Distance from the searched ZIP code in miles |
| `store_transfer` | boolean | True when a dealer group ships the car from a far store (shown in every radius search, like on the site) |
| `featured` | boolean | True for featured (sponsored) listings |
| `dealer_id` | integer | CarGurus dealer ID |
| `franchise_dealer` | boolean | True for franchise (brand) dealers |
| `seller_rating` | number | Details add-on: dealer rating on CarGurus (0-5) |
| `seller_review_count` | integer | Details add-on: number of dealer reviews |
| `equipment` | object | Details add-on: full equipment per category, e.g. {Safety: [ABS Brakes, ...], Technology: [...]} |
| `accident_count` | integer | Details add-on: accidents in the vehicle history report |
| `owner_count` | integer | Details add-on: previous owners in the vehicle history report |
| `dealer_fees` | array | Details add-on: dealer fees `[{name, amount}]`, e.g. Documentation Fee |
| `base_price` | number | Details add-on: price before dealer fees |
| `details_fetched` | boolean | True when the details add-on enriched this item |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: always false for CarGurus (no status field) |
| `price_history` | array | Price history add-on: price points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $0.75 per 1,000 car listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One listing enriched with its detail page: dealer description, all photos, full equipment list, accident and owner count, dealer fees and price before fees, dealer street address, coordinates, website and rating.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [CarGurus Car Listings Scraper](https://apify.com/carabain-digital/cargurus-listings)

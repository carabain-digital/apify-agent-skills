---
name: truecar-listings
description: "Scrape used and new car listings from TrueCar (truecar.com), the US car marketplace: price, TrueCar price rating (Excellent to Fair), mileage, VIN, specs, dealer and location. Search by ZIP, make, model, year, price, mileage, fuel and body. Add-ons: full details, price history. Runs the Apify Actor carabain-digital/truecar-listings. Use for tasks like: Used Toyota Camry near Los Angeles with TrueCar price ratings; Used EVs under $30,000 in the US: TrueCar price history."
---

# TrueCar Car Listings Scraper

Scrape used and new car listings from TrueCar (truecar.com), the US car marketplace: price, TrueCar price rating (Excellent to Fair), mileage, VIN, specs, dealer and location. Search by ZIP, make, model, year, price, mileage, fuel and body. Add-ons: full details, price history.

Runs on Apify as the Actor [`carabain-digital/truecar-listings`](https://apify.com/carabain-digital/truecar-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 5,555 car listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~truecar-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~truecar-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"location": "90210", "radius_km": 80, "make": "Toyota", "model": "Camry", "condition": "used", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/truecar-listings").call(run_input={"location": "90210", "radius_km": 80, "make": "Toyota", "model": "Camry", "condition": "used", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/truecar-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `location` | string | US ZIP code to search around (5 digits), as on TrueCar. Example: `10001` (New York), `90210`, `60601`. Leave empty to search all of the US. Ignored when you give Search URLs that carry their own location. |
| `radius_km` | integer | Distance from the ZIP code in kilometres (converted to miles: 80 km = 50 mi). Default 75 miles, maximum 8,000 km. TrueCar pads results with cars from farther dealers when the radius has few, each with its real `distance_mi`. Example: `160`. Needs a ZIP code. |
| `make` | string | Car make as TrueCar writes it. Example: `Toyota`, `Ford`, `Mercedes-Benz`, `Land Rover`. |
| `model` | string | Model of that make (needs Make). Example: `Camry`, `F-150`, `Model 3`, `CR-V`. |
| `condition` | string | `used` (default), `new`, or `any` (used cars first, then new cars). Example: `used`. Values: `used`, `new`, `any`. Default: `"used"`. |
| `min_year` | integer | Oldest model year. Example: `2018`. |
| `max_year` | integer | Newest model year. Example: `2023`. |
| `min_price` | integer | Lowest price in US dollars. Example: `10000`. |
| `max_price` | integer | Highest price in US dollars. Example: `25000`. |
| `max_mileage` | integer | Highest odometer reading in miles. Example: `60000`. |
| `fuel_type` | string | Fuel type. Example: `electric`. Values: `any`, `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`. Default: `"any"`. |
| `transmission` | string | Keep only automatic or manual cars. TrueCar has no such filter, so the Actor drops the others after reading each page (a rare value can need many pages). Example: `manual`. Values: `any`, `automatic`, `manual`. Default: `"any"`. |
| `body_type` | string | Body style. Example: `suv`. Values: `any`, `sedan`, `suv`, `pickup`, `hatchback`, `coupe`, `convertible`, `wagon`, `minivan`, `van`. Default: `"any"`. |
| `seller_type` | string | Keep only dealer or only private-seller listings. TrueCar lists dealers only, so `private` returns nothing. Example: `dealer`. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `sort` | string | Order of the results. TrueCar has no sort by listing date, so newest-first is not possible; use `listed_at` in the output. Example: `price_asc`. Values: `best_match`, `price_asc`, `price_desc`, `mileage_asc`, `mileage_desc`, `year_desc`, `year_asc`. Default: `"best_match"`. |
| `start_urls` | array | Search result pages copied from truecar.com; they replace the filters above. Example: `https://www.truecar.com/used-cars-for-sale/listings/toyota/camry/location-90210/`. Page number and sort order in the URL are ignored; without a ZIP in the URL all of the US is searched. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings. TrueCar serves at most 10,000 per search. Example: `100`. Default: `100`. |
| `includeDetails` | boolean | Also open each listing: dealer description, all photos, equipment highlights, warranties, accident and owner count from the vehicle history report, title flags, dealer fees, dealer phone, website and opening hours. Billed per listing as the Full details add-on. Default: `false`. |
| `trackHistory` | boolean | Keep the price of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `camry-nyc`. Default: `"cars-history"`. |

## Examples

**Used Toyota Camry near Los Angeles with TrueCar price ratings**: Scrape used Toyota Camry listings within 50 miles of Beverly Hills, Los Angeles (ZIP 90210) from TrueCar: price, TrueCar price rating, mileage, VIN, dealer and location. Export to JSON, CSV or Excel.

```json
{
  "location": "90210",
  "radius_km": 80,
  "make": "Toyota",
  "model": "Camry",
  "condition": "used",
  "limit": 100
}
```

**Used EVs under $30,000 in the US: TrueCar price history**: Scrape used electric cars under $30,000 from all of the US on TrueCar, cheapest first, and track their price over time: price drops, price rating, mileage and dealer. Run it daily to catch the best EV deals.

```json
{
  "fuel_type": "electric",
  "max_price": 30000,
  "sort": "price_asc",
  "limit": 200,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `title`, `price`, `price_rating`, `mileage`, `city`, `region`, `seller_name`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `truecar:<VIN>` |
| `source` | string | Short site name (truecar) |
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
| `price_rating` | string | TrueCar price rating compared with similar cars: excellent, great, good, fair (more levels possible); null for new cars |
| `msrp` | number | Original MSRP, new cars |
| `offer_price` | number | Price shown in the listing page's price box (Full details add-on), USD |
| `certified_pre_owned` | boolean | True for certified pre-owned (CPO) cars |
| `drivetrain` | string | Drive type, e.g. FWD, AWD, RWD, 4WD |
| `engine` | string | Engine as TrueCar shows it, e.g. 2.5L Inline-4 Gas |
| `transmission_text` | string | Transmission as TrueCar shows it, e.g. Automatic |
| `fuel_type_text` | string | Fuel type as TrueCar shows it, e.g. Gas, Hybrid, Electric |
| `interior_color` | string | Interior colour |
| `mpg_city` | number | City fuel economy (MPG) |
| `mpg_highway` | number | Highway fuel economy (MPG) |
| `mpg_combined` | number | Combined fuel economy (MPG) |
| `key_features` | array | Key features from the search card, e.g. Backup Camera, Apple CarPlay |
| `stock_number` | string | Dealer stock number |
| `distance_mi` | number | Distance from the searched ZIP code in miles (null when searching all of the US). TrueCar also lists cars from far dealers, with their real distance |
| `dealer_id` | string | TrueCar dealer ID |
| `accident_count` | integer | Accidents in the vehicle history report (search card; the Full details add-on keeps the report's number) |
| `owner_count` | integer | Previous owners in the vehicle history report |
| `clean_title` | boolean | True when the history shows no salvage, lemon or theft-recovery title |
| `equipment` | object | Details add-on: equipment highlights per category, e.g. {Safety: [Backup Camera, ...], Technology: [...]} |
| `warranties` | array | Details add-on: factory warranties `[{name, years, miles}]` |
| `dealer_fees` | array | Details add-on: dealer fees `[{name, amount}]`, e.g. Documentation Fee |
| `seller_hours` | object | Details add-on: dealer opening hours per weekday, e.g. {monday: 9:00am-8:00pm} |
| `details_fetched` | boolean | True when the details add-on enriched this item |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: always false for TrueCar (no status field) |
| `price_history` | array | Price history add-on: price points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $0.90 per 1,000 car listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One listing enriched with its detail page: dealer description, all photos, equipment highlights, warranties, accident and owner count, title flags, dealer fees, dealer phone, website and opening hours.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [TrueCar Car Listings Scraper](https://apify.com/carabain-digital/truecar-listings?fpr=4ijnom)

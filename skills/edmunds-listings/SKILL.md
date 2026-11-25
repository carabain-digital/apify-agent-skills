---
name: edmunds-listings
description: "Scrape used and new cars for sale from Edmunds inventory (edmunds.com/inventory), USA: price, Edmunds deal rating and price estimate, mileage, VIN, specs, dealer phone. Search by ZIP, make, model, year, price. Newest first. Add-ons: full details, price history. Listings only, no reviews. Runs the Apify Actor carabain-digital/edmunds-listings. Use for tasks like: Used Honda CR-V near Los Angeles on Edmunds; Used electric cars near Chicago: Edmunds price history."
---

# Edmunds Car Listings for Sale

Scrape used and new cars for sale from Edmunds inventory (edmunds.com/inventory), USA: price, Edmunds deal rating and price estimate, mileage, VIN, specs, dealer phone. Search by ZIP, make, model, year, price. Newest first. Add-ons: full details, price history. Listings only, no reviews.

Runs on Apify as the Actor [`carabain-digital/edmunds-listings`](https://apify.com/carabain-digital/edmunds-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 5,000 car listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~edmunds-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~edmunds-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"location": "90012", "make": "Honda", "model": "CR-V", "condition": "used", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/edmunds-listings").call(run_input={"location": "90012", "make": "Honda", "model": "CR-V", "condition": "used", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/edmunds-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `location` | string | US ZIP code to search around (5 digits). Example: `10001` (New York), `90210`, `60601`. Leave empty to search 500 miles around the middle of the US (Edmunds' largest radius). Also sets the area for search URLs. |
| `radius_km` | integer | Distance from the ZIP code in kilometres (converted to miles; Edmunds searches at most 500 miles). Default 50 miles (80 km). Only dealers within this distance are kept. Example: `160` for 100 miles. |
| `make` | string | Car make as Edmunds writes it. Example: `Toyota`, `Ford`, `Mercedes-Benz`, `Land Rover`. |
| `model` | string | Model of that make (needs Make). Example: `Camry`, `F-150`, `Model 3`, `CR-V`. |
| `condition` | string | `used` (includes Certified Pre-Owned, the Edmunds default), `new`, or `any` (used, certified and new together, newest first). Example: `used`. Values: `used`, `new`, `any`. Default: `"used"`. |
| `min_year` | integer | Oldest model year. Example: `2018`. |
| `max_year` | integer | Newest model year. Example: `2023`. |
| `min_price` | integer | Lowest price in US dollars. Example: `10000`. |
| `max_price` | integer | Highest price in US dollars. Example: `25000`. |
| `max_mileage` | integer | Highest odometer reading in miles. Example: `60000`. |
| `fuel_type` | string | Fuel type. Example: `electric`. Values: `any`, `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`. Default: `"any"`. |
| `transmission` | string | Transmission. Example: `manual`. Values: `any`, `automatic`, `manual`. Default: `"any"`. |
| `body_type` | string | Body style. Example: `suv`. Values: `any`, `sedan`, `suv`, `pickup`, `hatchback`, `coupe`, `convertible`, `wagon`, `minivan`, `van`. Default: `"any"`. |
| `seller_type` | string | Keep only dealer or only private-seller listings. Edmunds lists dealer inventory, so private is nearly always empty. Example: `dealer`. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `start_urls` | array | Search result pages copied from edmunds.com/inventory; their filters replace the ones above (the ZIP and radius still come from the fields). Example: `https://www.edmunds.com/inventory/srp.html?inventorytype=used%2Ccpo&make=honda&model=honda%7Ccr-v`. Page number and sort order are ignored; results come newest first. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `100`. Default: `100`. |
| `includeDetails` | boolean | Also open each listing: the dealer's complete description, full equipment list, dealer fees, MSRP and destination fee, dealer coordinates and the dealer's own page for the car. Charged per listing enriched. Example: `true`. Default: `false`. |
| `trackHistory` | boolean | Keep the price of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `camry-nyc`. Default: `"cars-history"`. |

## Examples

**Used Honda CR-V near Los Angeles on Edmunds**: Scrape the newest used Honda CR-V listings within 50 miles of Los Angeles (ZIP 90012) from Edmunds inventory: price, Edmunds deal rating and price estimate, mileage, VIN, dealer and phone. Export to JSON, CSV or Excel.

```json
{
  "location": "90012",
  "make": "Honda",
  "model": "CR-V",
  "condition": "used",
  "limit": 100
}
```

**Used electric cars near Chicago: Edmunds price history**: Scrape used electric cars under $35,000 around Chicago (ZIP 60601) from Edmunds inventory and track their price over time: price drops, Edmunds deal rating, mileage and dealer. Run it daily to catch the best EV deals.

```json
{
  "location": "60601",
  "radius_km": 80,
  "fuel_type": "electric",
  "max_price": 35000,
  "limit": 200,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `title`, `price`, `price_estimate`, `deal_rating`, `mileage`, `city`, `region`, `seller_name`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `edmunds:<VIN>` |
| `source` | string | Short site name (edmunds) |
| `url` | string | Public listing page on edmunds.com |
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
| `seller_url` | string | Details add-on: the dealer's own page for this car |
| `address` | string | Dealer address (city and state; full street address with the details add-on) |
| `city` | string | Dealer city |
| `postcode` | string | Dealer ZIP code |
| `region` | string | Dealer state, e.g. NJ |
| `country` | string | US |
| `lat` | number | Details add-on: dealer latitude |
| `lon` | number | Details add-on: dealer longitude |
| `listed_at` | string | Date the car was listed on CarGurus (YYYY-MM-DD, from days on market) |
| `image_urls` | array | Photo URLs (one without details, all with the details add-on) |
| `description` | string | Seller's comments (cut off after ~200 characters on the search card; complete with the Details add-on) |
| `deal_rating` | string | Edmunds deal rating: great, good, fair or above_market; null when Edmunds has no rating (too little data, many new cars) |
| `days_on_market` | integer | Days the car has been listed on CarGurus |
| `msrp` | number | Details add-on: manufacturer's suggested retail price incl. options, USD |
| `certified_pre_owned` | boolean | True for certified pre-owned (CPO) cars |
| `drivetrain` | string | e.g. All-Wheel Drive |
| `engine` | string | Engine as CarGurus shows it, e.g. 184 hp 2.4L I4 |
| `transmission_text` | string | Transmission as CarGurus shows it, e.g. 8-Speed Automatic |
| `fuel_type_text` | string | Fuel type as CarGurus shows it, e.g. Gasoline |
| `interior_color` | string | Interior colour |
| `mpg_city` | number | City fuel economy (MPG) |
| `mpg_highway` | number | Highway fuel economy (MPG) |
| `mpg_combined` | number | Combined fuel economy (MPG) |
| `options` | array | Top features from the search card, e.g. AWD/4WD |
| `stock_number` | string | Dealer stock number |
| `distance_mi` | number | Distance of the dealer from the searched ZIP code in miles |
| `dealer_id` | integer | Edmunds dealer (rooftop) ID |
| `equipment` | array | Details add-on: full equipment list as the dealer entered it |
| `accident_count` | integer | Details add-on: accidents in the vehicle history report |
| `owner_count` | integer | Details add-on: previous owners in the vehicle history report |
| `dealer_fees` | array | Details add-on: dealer fees `[{name, amount, included}]`, e.g. Documentation Fee |
| `details_fetched` | boolean | True when the details add-on enriched this item |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Details add-on: last change of the advertised price, USD (negative = drop) |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: always false for CarGurus (no status field) |
| `price_history` | array | Price history add-on: price points over your runs, `[{at, price, status}]`, last 20 |
| `price_estimate` | number | Edmunds' price estimate for this car in your area (what similar cars list for), USD |
| `price_vs_estimate` | number | Edmunds estimate minus the price: positive = listed below the estimate, negative = above |
| `deal_thresholds` | object | Price limits behind the deal rating: `{great_max, good_max, fair_max}` in USD |
| `savings_percent` | number | Percent below Edmunds' estimate (negative = above) |
| `in_transit` | boolean | True when the car is still being delivered to the dealer |
| `clean_title` | boolean | Vehicle history: clean title (only where Edmunds shows history) |
| `usage_type` | string | Vehicle history: previous use, e.g. Personal Use, Rental |
| `destination_fee` | number | Details add-on: factory destination fee, USD |

## Price

Pay per result: $1.00 per 1,000 car listings on the Apify Free plan, less on paid plans, plus $0.025 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One listing enriched with its Edmunds listing page: the dealer's complete description, full equipment list, dealer fees, MSRP and destination fee, last price change, dealer coordinates and the dealer's own page for the car.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [Edmunds Car Listings for Sale](https://apify.com/carabain-digital/edmunds-listings?fpr=4ijnom)

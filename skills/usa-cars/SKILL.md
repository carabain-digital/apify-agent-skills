---
name: usa-cars
description: "Search CarGurus, Autotrader, Cars.com, TrueCar, CarMax, Edmunds, Carvana and Craigslist at once for used and new cars in the US. One search by make, model, price, year, mileage and ZIP code; one dataset merged and deduplicated on VIN, newest first. Optional details and price history. Proxy included. Runs the Apify Actor carabain-digital/usa-cars. Use for tasks like: Used Toyota Camry under $25,000 from CarGurus and Autotrader; Used cars around Chicago with price history."
---

# USA Car Listings Scraper

Search CarGurus, Autotrader, Cars.com, TrueCar, CarMax, Edmunds, Carvana and Craigslist at once for used and new cars in the US. One search by make, model, price, year, mileage and ZIP code; one dataset merged and deduplicated on VIN, newest first. Optional details and price history. Proxy included.

Runs on Apify as the Actor [`carabain-digital/usa-cars`](https://apify.com/carabain-digital/usa-cars), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 5,000 listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~usa-cars/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~usa-cars/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"make": "Toyota", "model": "Camry", "condition": "used", "max_price": 25000, "location": "90001", "radius_miles": 50, "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/usa-cars").call(run_input={"make": "Toyota", "model": "Camry", "condition": "used", "max_price": 25000, "location": "90001", "radius_miles": 50, "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/usa-cars`.

## Input

| Field | Type | Description |
|---|---|---|
| `make` | string | Car make as the sites write it. Example: `Toyota`, `Ford`, `Mercedes-Benz`, `Land Rover`. Leave empty for all makes. A site that does not know the make is skipped (logged) instead of returning unrelated cars. |
| `model` | string | Model of that make (needs Make). Example: `Camry`, `F-150`, `CR-V`, `Model 3`. A site that does not know the model is skipped (logged). |
| `min_year` | integer | Earliest model year. Example: `2018`. |
| `max_year` | integer | Latest model year. Example: `2022`. |
| `min_price` | integer | Lowest asking price in US dollars. Example: `10000`. |
| `max_price` | integer | Highest asking price in US dollars. Example: `30000`. CarGurus filters on the price before dealer fees, so a few CarGurus prices can be slightly above it in fee-inclusive states such as California. |
| `max_mileage` | integer | Highest odometer reading in miles. Example: `60000`. |
| `fuel_type` | string | Fuel type. Example: `electric`. Autotrader has no flex fuel (E85) or CNG filter, so it is skipped for those two. Values: `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`, `ethanol`, `hydrogen`, `cng`. |
| `transmission` | string | Gearbox. Example: `automatic`. Values: `automatic`, `manual`. |
| `body_type` | string | Body type. Example: `pickup`. Values: `sedan`, `hatchback`, `wagon`, `suv`, `coupe`, `convertible`, `minivan`, `van`, `pickup`. |
| `condition` | string | New or used cars. Example: `used`. With `any`, CarGurus returns used cars first and new ones only after the used ones run out. CarMax sells used cars only: it is skipped for `new`. Values: `any`, `used`, `new`. Default: `"used"`. |
| `seller_type` | string | Dealer or private seller. Example: `dealer`. Almost every listing on these sites is from a dealer; CarGurus filters private sellers after the search and finds very few. Carvana and CarMax sell only their own cars: they are skipped for `private`. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `location` | string | Search around a 5-digit US ZIP code. Example: `10001` (New York), `60601` (Chicago), `90001` (Los Angeles). Leave empty to search the whole US (Craigslist cannot: it needs Craigslist area; Carvana ignores the location, it delivers nationwide). |
| `radius_miles` | integer | Distance around the ZIP code in miles. Example: `50`. CarGurus uses it as is; Autotrader rounds up to 10, 25, 50, 75, 100, 200, 300, 400, 500 miles (more = nationwide) and Cars.com to 10, 20, 30, 40, 50, 75, 100, 150, 200, 250, 500, 750, 1000. Wins over Radius (km). Default 50 miles. TrueCar and Edmunds take any distance, CarMax rounds up to 5, 25, 50, 75, 100, 250, 500; Carvana and Craigslist ignore it. |
| `radius_km` | integer | The same distance in km, for tools that pass the shared car-scraper field: converted to miles (80 km = 50 mi). Example: `80`. Ignored when Radius (miles) is set. |
| `craigslist_area` | string | Only for the site Craigslist, which searches one area (a city region) at a time and has no ZIP code or nationwide search. Use the area name as in `https://<area>.craigslist.org`. Example: `newyork`, `sfbay`, `chicago`, `losangeles`. Several areas, comma separated, are searched one after the other. Without it, a Location that is not a ZIP code (e.g. `chicago`) is used; with only a ZIP code Craigslist is skipped (logged). |
| `sites` | array | Which sites to search. Default: CarGurus and Autotrader (fast, seconds). Optional extra sources: Cars.com and Carvana run in a real browser behind Cloudflare (30 s to several minutes); CarMax and Edmunds clear Akamai in a stealth browser and only some runs get through (retried, then skipped); TrueCar is blocked on about 1 in 3 connections (retried); Craigslist is fast but needs a Craigslist area (see that field) and lists mostly private sellers. Example: `["cargurus", "autotrader", "truecar"]`. Default: `["cargurus", "autotrader"]`. |
| `limit` | integer | Cap on the merged dataset, newest first; every site gets a fair share. Example: `100`. Default: `100`. |
| `limit_per_site` | integer | Optional cap per site, to keep the mix balanced and Cars.com runs short. Default: Max listings. Example: `25`. |
| `includeDetails` | boolean | Also open each listing on its site: the dealer's description, all photos, equipment by category and more (CarGurus: accident and owner count, dealer fees, street address and coordinates; Autotrader: features, packages, warranties, open recalls, dealer hours; Cars.com: transmission, listing date, title and accident flags; TrueCar, CarMax, Edmunds, Carvana and Craigslist: description, photos, equipment, VIN and history or fee details where the site has them). Adds $1.20 per 1,000 listings that were actually enriched. Cars.com fetches details inside its browser run and is slow (1-30 s per listing): use a small Max listings per site with it. Default: `false`. |
| `trackHistory` | boolean | Keep a price history per listing in a named store in your own Apify account and add `first_seen_at`, `previous_price`, `price_change`, `price_change_pct` and `price_history` to every listing you have scraped before. Shared with our single-site car Actors. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Example: `golf-watch`. Default: `"cars-history"`. |

## Examples

**Used Toyota Camry under $25,000 from CarGurus and Autotrader**: The newest used Toyota Camry listings under $25,000 around Los Angeles from CarGurus and Autotrader in one list, deduplicated on VIN, with price, mileage, deal rating and dealer.

```json
{
  "make": "Toyota",
  "model": "Camry",
  "condition": "used",
  "max_price": 25000,
  "location": "90001",
  "radius_miles": 50,
  "limit": 100
}
```

**Used cars around Chicago with price history**: The newest used cars within 100 miles of Chicago from CarGurus and Autotrader, deduplicated, with price changes since your last run. Schedule it daily for a price-drop feed.

```json
{
  "condition": "used",
  "location": "60601",
  "radius_miles": 100,
  "limit": 500,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `source`, `title`, `year`, `mileage`, `price`, `deal_rating`, `city`, `region`, `seller_name`, `also_on`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `<source>:<site id>` (same IDs as the single-site Actors) |
| `source` | string | Site the item was taken from: cargurus, autotrader, carscom, truecar, carmax, edmunds, carvana or craigslist |
| `url` | string | Public listing page on the source site |
| `title` | string | Listing title, e.g. 2019 Honda CR-V LX AWD |
| `make` | string | Make, e.g. Toyota |
| `model` | string | Model, e.g. Camry |
| `variant` | string | Trim, e.g. LX AWD |
| `condition` | string | used or new |
| `price` | integer or number | Listed price in USD (CarGurus includes dealer fees where the state requires it). Null when the dealer shows no price |
| `price_currency` | string | USD |
| `price_type` | string | fixed, or on_request when there is no price |
| `mileage` | integer | Odometer reading |
| `mileage_unit` | string | mi (miles) |
| `year` | integer | Model year |
| `first_registration` | string | Always null: US listings only have the model year |
| `fuel_type` | string | petrol, diesel, electric, hybrid, plugin_hybrid, ethanol (flex fuel), cng, hydrogen or other |
| `transmission` | string | manual or automatic (Cars.com: details add-on only) |
| `body_type` | string | sedan, hatchback, wagon, suv, coupe, convertible, van, pickup, minivan or other |
| `power_kw` | integer or number | Engine power in kW (from hp) |
| `power_hp` | integer or number | Engine power in hp, when the site knows it (Cars.com: details add-on) |
| `engine_cc` | integer | Engine displacement in cc |
| `color` | string | Exterior colour |
| `doors` | integer | Number of doors |
| `seats` | integer | Always null: not published |
| `vin` | string | Vehicle identification number (17 characters) |
| `seller_type` | string | dealer or private |
| `seller_name` | string | Dealer name, or the private seller's first name |
| `seller_phone` | string | Dealer phone number (CarGurus, Autotrader; Cars.com hides it) |
| `seller_url` | string | Dealer website or page (details add-on on CarGurus and Cars.com) |
| `address` | string | Seller address (city and state; full street address with the details add-on) |
| `city` | string | Seller city |
| `postcode` | string | Seller ZIP code |
| `region` | string | Seller state, e.g. NJ |
| `country` | string | US |
| `lat` | number | Seller latitude (CarGurus details add-on, Autotrader) |
| `lon` | number | Seller longitude (CarGurus details add-on, Autotrader) |
| `listed_at` | string | Date the car was listed (YYYY-MM-DD; CarGurus and Autotrader from days on market, Cars.com only with the details add-on). The list is sorted on it, newest first |
| `image_urls` | array | Photo URLs (all photos with the details add-on) |
| `description` | string | Details add-on: the seller's description |
| `deal_rating` | string | The site's deal rating as it gives it (CarGurus: great_price ... overpriced; Autotrader and Cars.com: great, good, fair ...) |
| `imv_price` | number | CarGurus only: CarGurus Instant Market Value (IMV): what similar cars sell for, USD |
| `imv_difference` | number | CarGurus only: IMV minus price: positive = below market value, negative = above |
| `deal_score` | number | CarGurus only: CarGurus deal score behind the rating |
| `days_on_market` | integer | CarGurus only: Days the car has been listed on CarGurus |
| `total_price` | number | CarGurus only: CarGurus total price incl. fees where shown |
| `msrp` | number | Original MSRP |
| `certified_pre_owned` | boolean | True for certified pre-owned (CPO) cars (CarGurus, Cars.com) |
| `drivetrain` | string | Drivetrain, e.g. All-Wheel Drive (CarGurus, Cars.com) |
| `engine` | string | Engine as the site shows it, e.g. 184 hp 2.4L I4 |
| `transmission_text` | string | Transmission as the site shows it, e.g. 8-Speed Automatic |
| `fuel_type_text` | string | CarGurus only: Fuel type as CarGurus shows it, e.g. Gasoline |
| `interior_color` | string | Interior colour |
| `mpg_city` | number | City fuel economy (MPG) |
| `mpg_highway` | number | Highway fuel economy (MPG) |
| `mpg_combined` | number | CarGurus only: Combined fuel economy (MPG) |
| `options` | array | CarGurus only: Top options from the search card, e.g. Backup Camera |
| `stock_number` | string | Dealer stock number |
| `distance_mi` | number | Distance from the searched ZIP code in miles |
| `store_transfer` | boolean | CarGurus only: True when a dealer group ships the car from a far store (shown in every radius search, like on the site) |
| `featured` | boolean | CarGurus only: True for featured (sponsored) listings |
| `dealer_id` | integer or string | CarGurus: dealer ID (number); TrueCar: dealer ID (text) |
| `franchise_dealer` | boolean | CarGurus only: True for franchise (brand) dealers |
| `seller_rating` | number | Dealer rating (0-5): CarGurus details add-on, Cars.com |
| `seller_review_count` | integer | CarGurus only: Details add-on: number of dealer reviews |
| `equipment` | object or array | Details add-on: full equipment, per category (object: CarGurus, Cars.com) or as a list of strings (Edmunds) |
| `accident_count` | integer | CarGurus only: Details add-on: accidents in the vehicle history report |
| `owner_count` | integer | CarGurus only: Details add-on: previous owners in the vehicle history report |
| `dealer_fees` | array | CarGurus only: Details add-on: dealer fees `[{name, amount}]`, e.g. Documentation Fee |
| `base_price` | number | CarGurus only: Details add-on: price before dealer fees |
| `details_fetched` | boolean | True when the details add-on enriched this item |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: always false (car sites have no status field) |
| `price_history` | array | Price history add-on: price points over your runs, `[{at, price, status}]`, last 20 |
| `certified` | boolean | Autotrader only: True for manufacturer or third-party certified pre-owned cars |
| `trim` | string | Autotrader only: Trim name as the site shows it |
| `drive_type` | string | Autotrader only: Drive type, e.g. All wheel drive |
| `engine_displacement_l` | number | Autotrader only: Engine displacement in litres |
| `days_on_site` | integer | Autotrader only: Days the car has been listed on Autotrader |
| `kbb_fair_price` | number | Autotrader only: Kelley Blue Book Fair Purchase Price in USD |
| `kbb_fair_price_low` | number | Autotrader only: Low end of the KBB fair price range |
| `kbb_fair_price_high` | number | Autotrader only: High end of the KBB fair price range |
| `price_reduced` | boolean | Autotrader only: True when the dealer recently cut the price |
| `site_price_history` | array | The site's own price changes for this listing, `[{date, price}]` (Autotrader; Cars.com details add-on) |
| `vehicle_history` | array | Autotrader only: Vehicle history badges, e.g. NO_ACCIDENTS_REPORTED, ONE_OWNER, NO_SALVAGE_TITLE |
| `dealer_rating` | number | Autotrader only: Dealer review score (0-5) |
| `dealer_review_count` | integer | Autotrader only: Number of dealer reviews |
| `features` | object | Autotrader only: Details add-on: equipment by category (exterior, interior, mechanical, safety, technology, other) |
| `highlights` | array | Autotrader only: Details add-on: the key features the site highlights, e.g. Backup Camera, Apple CarPlay |
| `packages` | array | Autotrader only: Details add-on: option packages |
| `warranties` | object or array | Details add-on. Autotrader: original warranties as an object, e.g. {basic: '3 Years/36,000 Miles'}; TrueCar: a list `[{name, years, miles}]` |
| `open_recalls` | integer | Autotrader only: Details add-on: number of open safety recalls |
| `video_urls` | array | Autotrader only: Details add-on: walk-around video URLs |
| `seller_phones` | array | Autotrader only: Details add-on: every phone number of the dealer |
| `dealer_hours` | object | Autotrader only: Details add-on: opening hours per weekday |
| `badges` | array | Cars.com only: Badges on the card, e.g. Great Deal, Price Drop, Certified Pre-Owned, award finalist |
| `monthly_payment_estimate` | number | Cars.com only: Cars.com's estimated monthly payment in USD (72 months, 7% APR, $0 down) |
| `photo_count` | number | Cars.com only: Number of photos of the listing |
| `mpg` | string or number | Carvana: combined MPG (number); Cars.com, details add-on: fuel economy as text, e.g. 25-33 mpg |
| `vehicle_reports` | object | Cars.com only: Details add-on: dealer-verified report facts (e.g. CPO inspection, warranty, roadside assistance) |
| `clean_title` | boolean | Cars.com only: Details add-on: clean title (when the site knows it) |
| `no_accidents` | boolean | Cars.com only: Details add-on: no reported accidents (when the site knows it) |
| `single_owner` | boolean | Cars.com only: Details add-on: one previous owner (when the site knows it) |
| `personal_use_only` | boolean | Cars.com only: Details add-on: personal use only (when the site knows it) |
| `seller_id` | string | Cars.com only: Details add-on: cars.com dealer id |
| `seller_website` | string | Cars.com only: Details add-on: dealer's own website |
| `price_rating` | string | TrueCar only: TrueCar price rating compared with similar cars: excellent, great, good, fair (more levels possible); null for new cars |
| `offer_price` | number | TrueCar only: Price shown in the listing page's price box (Full details add-on), USD |
| `key_features` | array | TrueCar only: Key features from the search card, e.g. Backup Camera, Apple CarPlay |
| `seller_hours` | object | TrueCar only: Details add-on: dealer opening hours per weekday, e.g. {monday: 9:00am-8:00pm} |
| `area` | string | Craigslist only: Craigslist area (subdomain) searched, e.g. `sfbay` |
| `subarea` | string | Craigslist only: Sub-area abbreviation inside the area, e.g. `sby` (south bay) |
| `post_id` | integer | Craigslist only: Craigslist post id |
| `uuid` | string | Craigslist only: Craigslist listing key used in the URL |
| `cylinders` | string | Craigslist only: Details add-on: e.g. `6 cylinders` |
| `title_status` | string | Craigslist only: Details add-on: clean, salvage, rebuilt, parts only, lien or missing |
| `body_type_original` | string | Craigslist only: Details add-on: Craigslist's own body type (bus, offroad, truck, ...) |
| `make_model_text` | string | Craigslist only: Details add-on: the make and model as the seller typed them |
| `updated_at` | string | Craigslist only: Details add-on: last update time, UTC ISO 8601 |
| `kbb_value` | number | Carvana only: Kelley Blue Book value Carvana shows, USD |
| `price_vs_kbb` | number | Carvana only: Price minus KBB value (negative = below KBB) |
| `price_updated_at` | string | Carvana only: When the price last changed on carvana.com (ISO 8601) |
| `trim_kbb` | string | Carvana only: KBB trim name, e.g. SLE Sport Utility 4D |
| `purchase_pending` | boolean | Carvana only: True when someone has this car in checkout |
| `facility_id` | number | Carvana only: Carvana facility id the car is stored at |
| `in_service_date` | string | Carvana only: Details add-on: date the car first went into service (YYYY-MM-DD) |
| `accident_reported` | boolean | Carvana only: Details add-on: Autocheck reports an accident |
| `prior_use` | array | Carvana and CarMax: Details add-on: previous use, e.g. Rental, Lease |
| `number_of_keys` | number | Carvana only: Details add-on: number of keys |
| `recalls` | array | Carvana only: Details add-on: open recalls |
| `installed_options` | array | Carvana only: Details add-on: factory options and packages installed |
| `inspection` | object | Carvana only: Details add-on: Carvana's inspection summary `{inspected_on, points, passed, fixed_or_replaced}` |
| `warranty` | object or array | Details add-on. Carvana: remaining manufacturer warranty in months and miles (object); CarMax: warranties `[{type, years, miles}]` (list) |
| `dimensions` | object | Carvana only: Details add-on: length, width, height, wheelbase, curb weight, towing and trunk capacity |
| `original_price` | number | CarMax only: Price before the latest price drop, when the car had one |
| `price_drop` | boolean | CarMax only: True when the price was recently lowered |
| `new_arrival` | boolean | CarMax only: True for a car that just arrived on the lot |
| `body_style` | string | CarMax only: Body style as CarMax shows it, e.g. 4D Sedan |
| `store_id` | integer | CarMax only: CarMax store number |
| `shipping_fee` | number | CarMax only: Shipping fee in USD when the car is at a far store (0 = free); null when it is at the searched store |
| `shipping_text` | string | CarMax only: Shipping/test-drive text, e.g. $349 Shipping |
| `reserved` | boolean | CarMax only: True when the car is reserved by another customer |
| `generation` | string | CarMax only: Details add-on: model generation, e.g. Ninth generation |
| `price_estimate` | number | Edmunds only: Edmunds' price estimate for this car in your area (what similar cars list for), USD |
| `price_vs_estimate` | number | Edmunds only: Edmunds estimate minus the price: positive = listed below the estimate, negative = above |
| `deal_thresholds` | object | Edmunds only: Price limits behind the deal rating: `{great_max, good_max, fair_max}` in USD |
| `savings_percent` | number | Edmunds only: Percent below Edmunds' estimate (negative = above) |
| `in_transit` | boolean | Edmunds only: True when the car is still being delivered to the dealer |
| `usage_type` | string | Edmunds only: Vehicle history: previous use, e.g. Personal Use, Rental |
| `destination_fee` | number | Edmunds only: Details add-on: factory destination fee, USD |
| `also_on` | array | Other sites that carry the same car (same VIN, else same make, year, mileage and price); empty when only one does |

## Price

Pay per result: $1.00 per 1,000 listings on the Apify Free plan, less on paid plans, plus $0.025 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- Full details: $1.20 per 1,000. Opens each listing on its site: description, all photos, equipment, history flags, dealer details and more. Charged only for listings that were enriched.
- Price history: $1.00 per 1,000. One listing that already had a history in your named store, annotated with previous price, price change and the price history. The first run of a listing is free.

Store page, reviews and full documentation: [USA Car Listings Scraper](https://apify.com/carabain-digital/usa-cars)

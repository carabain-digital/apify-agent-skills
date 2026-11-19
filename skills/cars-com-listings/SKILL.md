---
name: cars-com-listings
description: "Scrape new and used car listings from Cars.com, a top US car marketplace: price, mileage, year, VIN, deal rating, dealer or private seller, city and photos. Filter by make, model, year, price, mileage, fuel and ZIP radius. Add-ons: full details, price history. Runs the Apify Actor carabain-digital/cars-com-listings. Use for tasks like: Used Toyota RAV4 under $30,000 near Chicago on Cars.com; Used electric cars near Los Angeles with price history."
---

# Cars.com Car Listings Scraper

Scrape new and used car listings from Cars.com, a top US car marketplace: price, mileage, year, VIN, deal rating, dealer or private seller, city and photos. Filter by make, model, year, price, mileage, fuel and ZIP radius. Add-ons: full details, price history.

Runs on Apify as the Actor [`carabain-digital/cars-com-listings`](https://apify.com/carabain-digital/cars-com-listings), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 5,000 car listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~cars-com-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~cars-com-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"make": "Toyota", "model": "RAV4", "condition": "used", "max_price": 30000, "location": "60601", "radius_miles": 50, "limit": 200}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/cars-com-listings").call(run_input={"make": "Toyota", "model": "RAV4", "condition": "used", "max_price": 30000, "location": "60601", "radius_miles": 50, "limit": 200})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/cars-com-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `make` | string | Car make as Cars.com writes it. Example: `Toyota`, `Ford`, `Mercedes-Benz`, `Land Rover`. Leave empty for all makes. An unknown make stops the run with an error instead of returning unrelated cars. |
| `model` | string | Model of that make as Cars.com writes it. Example: `Camry`, `F-150`, `C-Class`, `Range Rover Sport`. Needs a make. If a model is not found, run the search on cars.com and paste its URL into Search URLs. |
| `min_year` | integer | Earliest model year. Example: `2018`. |
| `max_year` | integer | Latest model year. Example: `2022`. |
| `min_price` | integer | Lowest asking price in US dollars. Example: `10000`. |
| `max_price` | integer | Highest asking price in US dollars. Example: `30000`. |
| `max_mileage` | integer | Highest odometer reading in miles. Example: `60000`. |
| `fuel_type` | string | Fuel type. Example: `electric`. Values: `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`, `ethanol`, `hydrogen`, `cng`. |
| `transmission` | string | Gearbox. Example: `automatic`. Values: `automatic`, `manual`. |
| `body_type` | string | Body type. Example: `pickup`. Values: `sedan`, `hatchback`, `wagon`, `suv`, `coupe`, `convertible`, `minivan`, `van`, `pickup`. |
| `condition` | string | New or used cars. Example: `used`. Values: `any`, `used`, `new`. Default: `"any"`. |
| `seller_type` | string | Dealer or private seller. Example: `private`. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `location` | string | Search around a 5-digit US ZIP code. Example: `60601` (Chicago), `90001` (Los Angeles). Leave empty to search the whole US. |
| `radius_miles` | integer | Distance around the ZIP code in miles, the unit Cars.com uses. Example: `50`. Rounded up to the site's options (10, 20, 30, 40, 50, 75, 100, 150, 200, 250, 500, 750, 1000; more = nationwide). Wins over Radius (km). Default 50 miles. |
| `radius_km` | integer | The same distance in km, for tools that pass the shared car-scraper field: converted to miles (80 km = 50 mi). Example: `80`. Ignored when Radius (miles) is set. |
| `start_urls` | array | Search result pages copied from cars.com; they replace the filters above. Example: `https://www.cars.com/shopping/results/?stock_type=used&makes[]=toyota&models[]=toyota-rav4&zip=60601&maximum_distance=50`. All filters and the sort order in the URL are kept (newest first when the URL has none); the page number is ignored. Only /shopping/results/ pages work. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `500`. Cars.com shows at most 10,000 cars per search: split bigger searches by make, price or ZIP. Default: `100`. |
| `includeDetails` | boolean | Also open each listing: seller's notes, all photos, transmission, engine, horsepower, MPG, interior colour, all features by category, VIN and stock number, listing date and the site's price changes, dealer address, website and cars.com page, title/accident/owner flags. Adds $3.00 per 1,000 listings. About 1 s extra per listing, up to 30 s when Cloudflare re-checks the pages: use it with a modest Max listings (a run stops adding details after 45 minutes). Default: `false`. |
| `trackHistory` | boolean | Keep the price of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct and price_history. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Example: `golf-watch`. Default: `"cars-history"`. |

## Examples

**Used Toyota RAV4 under $30,000 near Chicago on Cars.com**: Scrape the newest used Toyota RAV4 listings under $30,000 within 50 miles of Chicago from cars.com: price, deal rating, mileage, year, VIN, dealer and city. Export to JSON, CSV or Excel.

```json
{
  "make": "Toyota",
  "model": "RAV4",
  "condition": "used",
  "max_price": 30000,
  "location": "60601",
  "radius_miles": 50,
  "limit": 200
}
```

**Used electric cars near Los Angeles with price history**: Track used electric cars within 100 miles of Los Angeles on cars.com over time: asking price, price drops, deal rating, mileage, year and dealer. Run it daily to catch price cuts.

```json
{
  "fuel_type": "electric",
  "condition": "used",
  "location": "90001",
  "radius_miles": 100,
  "limit": 500,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `title`, `year`, `mileage`, `fuel_type`, `price`, `deal_rating`, `city`, `region`, `seller_type`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `carscom:<cars.com listing uuid>` |
| `source` | string | Short site name: carscom |
| `url` | string | Public listing URL on cars.com |
| `title` | string | Make, model and trim, e.g. Volkswagen Golf 1.5 TSI Life |
| `make` | string | Make, e.g. BMW |
| `model` | string | Model, e.g. X1 |
| `variant` | string | Trim / version as the seller wrote it, e.g. xDrive28i M Sport |
| `condition` | string | used or new (certified pre-owned cars are used: see certified_pre_owned) |
| `price` | integer | Asking price in US dollars (with dealer fees where the state requires them) |
| `price_currency` | string | EUR |
| `price_type` | string | fixed, negotiable (details add-on) or on_request |
| `mileage` | integer | Odometer in miles |
| `mileage_unit` | string | Always mi |
| `year` | integer | Model year |
| `first_registration` | string | Not published by cars.com (null) |
| `fuel_type` | string | petrol, diesel, electric, hybrid, plugin_hybrid, ethanol (flex fuel), hydrogen, cng or other |
| `transmission` | string | Details add-on: automatic (incl. CVT) or manual |
| `body_type` | string | sedan, hatchback, wagon, suv, coupe, convertible, van, pickup, minivan or other |
| `power_kw` | integer | Details add-on: power in kW (from the horsepower, when the seller gives it) |
| `power_hp` | integer | Details add-on: horsepower, when the seller gives it |
| `engine_cc` | integer | Details add-on: engine displacement in cc (from the litres, e.g. 2.5L = 2500) |
| `color` | string | Exterior colour (search: generic colour; details add-on: the maker's colour name) |
| `doors` | integer | Details add-on: number of doors |
| `seats` | integer | Details add-on: number of seats |
| `vin` | string | Vehicle identification number: AutoScout24 does not publish it, always null |
| `seller_type` | string | dealer or private |
| `seller_name` | string | Dealer name, or the private seller's first name |
| `seller_phone` | string | Not in the page (cars.com shows a call-tracking number by script): null |
| `seller_url` | string | Details add-on: dealer page on cars.com |
| `address` | string | Details add-on: dealer street address |
| `city` | string | City of the dealer or private seller |
| `postcode` | string | ZIP code of the seller |
| `region` | string | US state code, e.g. TX |
| `country` | string | Always US |
| `lat` | number | Not published (null) |
| `lon` | number | Not published (null) |
| `listed_at` | string | Details add-on: date the car was listed (YYYY-MM-DD, from the site's price history) |
| `image_urls` | array | Photo URLs: up to 6 from the search card, all photos with the details add-on |
| `description` | string | Details add-on: seller's notes |
| `certified_pre_owned` | boolean | Certified pre-owned (CPO) car |
| `deal_rating` | string | Cars.com deal rating vs the market: great, good or fair (null when not rated) |
| `badges` | array | Badges on the card, e.g. Great Deal, Price Drop, Certified Pre-Owned, award finalist |
| `msrp` | number | Manufacturer's suggested retail price (new cars) |
| `monthly_payment_estimate` | number | Cars.com's estimated monthly payment in USD (72 months, 7% APR, $0 down) |
| `drivetrain` | string | Drivetrain, e.g. All-wheel Drive |
| `distance_mi` | number | Distance from the searched ZIP code in miles |
| `seller_rating` | number | Dealer's average review stars (1-5) |
| `photo_count` | number | Number of photos of the listing |
| `details_fetched` | boolean | True when the Full details add-on enriched this item |
| `interior_color` | string | Details add-on: interior colour |
| `engine` | string | Details add-on: engine as the seller describes it, e.g. 2.5L I-4 DOHC |
| `transmission_text` | string | Details add-on: transmission as the seller describes it, e.g. 8-Speed Automatic |
| `mpg` | string | Details add-on: fuel economy, e.g. 25-33 mpg |
| `stock_number` | string | Details add-on: dealer stock number |
| `equipment` | object | Details add-on: features by category (Convenience, Entertainment, Exterior, Safety, Seating) |
| `vehicle_reports` | object | Details add-on: dealer-verified report facts (e.g. CPO inspection, warranty, roadside assistance) |
| `clean_title` | boolean | Details add-on: clean title (when the site knows it) |
| `no_accidents` | boolean | Details add-on: no reported accidents (when the site knows it) |
| `single_owner` | boolean | Details add-on: one previous owner (when the site knows it) |
| `personal_use_only` | boolean | Details add-on: personal use only (when the site knows it) |
| `site_price_history` | array | Details add-on: the listing's price changes on cars.com, `[{date, change, price}]` |
| `seller_id` | string | Details add-on: cars.com dealer id |
| `seller_website` | string | Details add-on: dealer's own website |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: true when the status changed |
| `price_history` | array | Price history add-on: price points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $1.00 per 1,000 car listings on the Apify Free plan, less on paid plans, plus $0.025 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- Full details: $3.00 per 1,000. One car listing enriched with its cars.com listing page: seller's notes, all photos, transmission, engine, horsepower, MPG, interior colour, features by category, stock number, listing date and price changes on the site, dealer address, website and cars.com page, title/accident/owner flags.
- Price history: $1.00 per 1,000. One car listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [Cars.com Car Listings Scraper](https://apify.com/carabain-digital/cars-com-listings)

---
name: autowereld-listings
description: "Scrape used car listings from AutoWereld (autowereld.nl), the Dutch car marketplace: price, mileage, year, fuel, gearbox, body, dealer and photos. Filter by make, model, price, year, km, fuel, location. Add-ons: full details with licence plate, price history. Runs the Apify Actor carabain-digital/autowereld-listings. Use for tasks like: Newest Volkswagen Golf listings on AutoWereld; Electric cars near Utrecht with price history on AutoWereld."
---

# AutoWereld Car Listings Scraper

Scrape used car listings from AutoWereld (autowereld.nl), the Dutch car marketplace: price, mileage, year, fuel, gearbox, body, dealer and photos. Filter by make, model, price, year, km, fuel, location. Add-ons: full details with licence plate, price history.

Runs on Apify as the Actor [`carabain-digital/autowereld-listings`](https://apify.com/carabain-digital/autowereld-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 5,000 car listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~autowereld-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~autowereld-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"make": "Volkswagen", "model": "Golf", "condition": "used", "max_price": 20000, "limit": 200}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/autowereld-listings").call(run_input={"make": "Volkswagen", "model": "Golf", "condition": "used", "max_price": 20000, "limit": 200})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/autowereld-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `make` | string | Car make as AutoWereld names it. Example: `Volkswagen`, `Mercedes-Benz`, `BMW`, `Land Rover`. Leave empty for all makes. |
| `model` | string | Model of that make (needs `make`). Example: `Golf`, `3 Serie`, `C-Klasse`, `Model 3`. Leave empty for all models. |
| `min_year` | integer | Oldest build year. Example: `2016`. |
| `max_year` | integer | Newest build year. Example: `2022`. |
| `min_price` | integer | Lowest asking price in euro, VAT included. Example: `5000`. |
| `max_price` | integer | Highest asking price in euro, VAT included. Example: `25000`. |
| `max_mileage` | integer | Highest odometer reading in km. Example: `150000`. |
| `fuel_type` | string | Fuel. `hybrid` and `plugin_hybrid` both use AutoWereld's Hybrid filter (plug-ins included). Example: `electric`. Values: `any`, `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`, `lpg`, `cng`, `hydrogen`. Default: `"any"`. |
| `transmission` | string | Gearbox. Example: `automatic`. Values: `any`, `manual`, `automatic`. Default: `"any"`. |
| `body_type` | string | Body style. Example: `suv`. Values: `any`, `sedan`, `hatchback`, `wagon`, `suv`, `coupe`, `convertible`, `van`, `pickup`, `minivan`, `other`. Default: `"any"`. |
| `condition` | string | AutoWereld lists used cars only; `new` is refused. Example: `used`. Values: `any`, `used`. Default: `"any"`. |
| `seller_type` | string | Dealer or private seller. Example: `dealer`. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `location` | string | Dutch postcode or town for a radius search (see `radius_km`). Example: `3511`, `Amersfoort`. Leave empty for all of the Netherlands. Ignored when you give search URLs. |
| `radius_km` | integer | Distance around the postcode or town, in km. AutoWereld offers 0, 10, 15, 20, 25, 30, 40, 50, 75, 100, 150 and 200: the next higher one is used. Default 10. Example: `25`. |
| `start_urls` | array | Search result pages copied from autowereld.nl; they replace the filters above. Example: `https://www.autowereld.nl/volkswagen/golf/?bjvan=2019`. Filters in the URL are kept; results come newest first unless the URL sorts otherwise (`srtr=`). Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many cars (newest first). Example: `100`. Default: `100`. |
| `includeDetails` | boolean | Also open each car's page: licence plate (kenteken), date listed, dealer description, equipment by category, all photos, dealer address, phone and coordinates, seats, power, APK (MOT) expiry, consumption, CO2, torque, drive, damage history. Adds $1.00 per 1,000 cars on top of the base price. One extra request per car (about 25 s per 100 cars). Default: `false`. |
| `trackHistory` | boolean | Keep the price and status of every car in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 cars that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `golf-utrecht`. Default: `"cars-history"`. |

## Examples

**Newest Volkswagen Golf listings on AutoWereld**: Scrape the newest used Volkswagen Golf cars on AutoWereld (autowereld.nl) up to EUR 20,000: price, mileage, year, fuel, gearbox, dealer and photos. Add full details for the licence plate. Export to JSON, CSV or Excel.

```json
{
  "make": "Volkswagen",
  "model": "Golf",
  "condition": "used",
  "max_price": 20000,
  "limit": 200
}
```

**Electric cars near Utrecht with price history on AutoWereld**: Scrape electric cars for sale within 25 km of Utrecht from AutoWereld (autowereld.nl) and track their prices over time: price drops, mileage, dealer, licence plate and full details. Run it daily to catch price cuts.

```json
{
  "fuel_type": "electric",
  "location": "Utrecht",
  "radius_km": 25,
  "limit": 300,
  "includeDetails": true,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `title`, `price`, `year`, `mileage`, `fuel_type`, `city`, `seller_name`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `autowereld:<listing id>` |
| `source` | string | Short site name (autowereld) |
| `url` | string | Car page on autowereld.nl |
| `title` | string | Make, model and variant |
| `make` | string | Make, e.g. Volkswagen |
| `model` | string | Model, e.g. Golf |
| `variant` | string | Variant / trim, e.g. 1.5 TSI Life |
| `condition` | string | used or new |
| `price` | number | Asking price in EUR, VAT included; null when on request |
| `price_currency` | string | EUR |
| `price_type` | string | fixed or on_request |
| `mileage` | integer | Odometer reading |
| `mileage_unit` | string | km |
| `year` | integer | Build year |
| `first_registration` | string | First registration, YYYY-MM |
| `fuel_type` | string | petrol, diesel, electric, hybrid, plugin_hybrid, lpg, cng, hydrogen or other |
| `transmission` | string | manual or automatic |
| `body_type` | string | sedan, hatchback, wagon, suv, coupe, convertible, van, pickup, minivan or other |
| `power_kw` | integer | Power in kW |
| `power_hp` | integer | Power in hp (pk) |
| `engine_cc` | integer | Engine displacement in cc |
| `color` | string | Colour (Dutch, lower case: zwart, grijs, wit ...) |
| `doors` | integer | Number of doors |
| `seats` | integer | Details add-on: number of seats |
| `vin` | string | Always null: AutoWereld shows the licence plate, not the VIN |
| `seller_type` | string | `dealer` or `private`. Search results only know it when you filter by seller type; the details add-on always fills it |
| `seller_name` | string | Dealer name |
| `seller_phone` | string | Dealer phone number |
| `seller_url` | string | Seller's page on AutoWereld (details add-on; dealers only) |
| `address` | string | Details add-on: dealer street, postcode and town |
| `city` | string | Dealer's town |
| `postcode` | string | Details add-on: dealer postcode, e.g. 4157JA |
| `region` | string | Dealer's province, e.g. Gelderland |
| `country` | string | NL |
| `lat` | number | Details add-on: dealer latitude |
| `lon` | number | Details add-on: dealer longitude |
| `listed_at` | string | Details add-on: when the car was put online, local time `YYYY-MM-DDTHH:MM:SS` |
| `image_urls` | array | All photo URLs |
| `description` | string | Details add-on: the dealer's text block (technical data, condition, delivery packages; Dutch) |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `license_plate` | string | Dutch licence plate (kenteken), e.g. N-597-GF. Details add-on only: search cards do not show it |
| `vat_deductible` | boolean | Details add-on: VAT deductible (false for margin cars) |
| `energy_label` | string | Energy label A-G |
| `seller_id` | string | AutoWereld seller id (details add-on) |
| `details_fetched` | boolean | True when the details add-on enriched this item |
| `equipment` | array | Details add-on: equipment by category, `[{category, options}]` |
| `gears` | integer | Details add-on: number of gears |
| `drive` | string | Details add-on: front, rear or 4x4 |
| `cylinders` | integer | Details add-on: cylinders |
| `torque_nm` | integer | Details add-on: torque in Nm |
| `top_speed_kmh` | integer | Details add-on: top speed |
| `acceleration_0_100_s` | number | Details add-on: 0-100 km/h in seconds |
| `consumption_km_per_l` | number | Details add-on: combined consumption in km per litre (Dutch '1 op N') |
| `co2_g_km` | integer | Details add-on: CO2 g/km |
| `emission_class` | string | Details add-on: e.g. Euro 6 |
| `empty_weight_kg` | integer | Details add-on: empty weight |
| `towing_capacity_kg` | integer | Details add-on: braked towing capacity |
| `apk_expires` | string | Details add-on: APK (MOT) valid until, YYYY-MM-DD |
| `maintenance_booklet` | boolean | Details add-on: maintenance booklet present |
| `updated_at` | string | Details add-on: when the dealer last changed the listing |
| `first_seen_at` | string | Price history add-on: when this car was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: status changed since the previous run |
| `price_history` | array | Price history add-on: `[{at, price, status}]`, last 20 |
| `warranty` | string | Warranty label shown on the card or page, e.g. BOVAG garantie (several: comma separated) |
| `damage_history` | string | Details add-on: damage history as the dealer states it, e.g. Ja (gerepareerd) |
| `nap` | boolean | Details add-on: true when the odometer is NAP-checked |

## Price

Pay per result: $1.00 per 1,000 car listings on the Apify Free plan, less on paid plans, plus $0.05 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One car enriched with its AutoWereld page: licence plate (kenteken), date listed, dealer description, full equipment list by category, all photos, dealer address, phone and coordinates, seats, power, APK expiry, consumption, CO2, torque, drive, damage history.
- Price history: $1.00 per 1,000. One car that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a car is free.

Store page, reviews and full documentation: [AutoWereld Car Listings Scraper](https://apify.com/carabain-digital/autowereld-listings?fpr=4ijnom)

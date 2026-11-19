---
name: autotrack-listings
description: "Scrape car listings from AutoTrack (autotrack.nl), the Dutch car marketplace: used and new dealer cars with price, mileage, year, fuel, gearbox, power, licence plate, photos and dealer phone. Filter by make, model, price, year, km, fuel, location. Add-ons: full details, price history. Runs the Apify Actor carabain-digital/autotrack-listings. Use for tasks like: Newest Volkswagen Golf listings on AutoTrack; Electric cars near Utrecht with price history on AutoTrack."
---

# AutoTrack Car Listings Scraper

Scrape car listings from AutoTrack (autotrack.nl), the Dutch car marketplace: used and new dealer cars with price, mileage, year, fuel, gearbox, power, licence plate, photos and dealer phone. Filter by make, model, price, year, km, fuel, location. Add-ons: full details, price history.

Runs on Apify as the Actor [`carabain-digital/autotrack-listings`](https://apify.com/carabain-digital/autotrack-listings), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 10,000 car listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~autotrack-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~autotrack-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"make": "Volkswagen", "model": "Golf", "condition": "used", "max_price": 20000, "limit": 200}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/autotrack-listings").call(run_input={"make": "Volkswagen", "model": "Golf", "condition": "used", "max_price": 20000, "limit": 200})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/autotrack-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `make` | string | Car make as AutoTrack names it. Example: `Volkswagen`, `Mercedes-Benz`, `BMW`, `Land Rover`. Leave empty for all makes. |
| `model` | string | Model of that make (needs `make`). Example: `Golf`, `3 Serie`, `C-Klasse`, `Model 3`. Leave empty for all models. |
| `min_year` | integer | Oldest build year. Example: `2016`. |
| `max_year` | integer | Newest build year. Example: `2022`. |
| `min_price` | integer | Lowest asking price in euro, VAT included. Example: `5000`. |
| `max_price` | integer | Highest asking price in euro, VAT included. Example: `25000`. |
| `max_mileage` | integer | Highest odometer reading in km. Example: `150000`. |
| `fuel_type` | string | Fuel. `hybrid` = every hybrid (AutoTrack's Hybride filter, plug-ins included); `plugin_hybrid` = plug-in hybrids only. Example: `electric`. Values: `any`, `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`, `lpg`, `cng`, `hydrogen`. Default: `"any"`. |
| `transmission` | string | Gearbox. Example: `automatic`. Values: `any`, `manual`, `automatic`. Default: `"any"`. |
| `body_type` | string | Body style. Example: `suv`. Values: `any`, `sedan`, `hatchback`, `wagon`, `suv`, `coupe`, `convertible`, `van`, `pickup`, `minivan`, `other`. Default: `"any"`. |
| `condition` | string | Used cars (occasions) or new cars. Example: `used`. Values: `any`, `used`, `new`. Default: `"any"`. |
| `seller_type` | string | AutoTrack only lists dealer cars, so `private` returns nothing. Example: `dealer`. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `location` | string | Dutch postcode, town or province. A postcode or town gives a radius search (see `radius_km`); a province (`Noord-Brabant`, `Gelderland` ...) filters on the dealer's province. `Utrecht` and `Groningen` mean the province, or the city when you also set `radius_km`. Example: `3511`, `Amersfoort`, `Noord-Holland`. Leave empty for all of the Netherlands. Ignored when you give search URLs. |
| `radius_km` | integer | Distance around the postcode or town, in km. Default 10. Example: `25`. |
| `start_urls` | array | Search result pages copied from autotrack.nl; they replace the filters above. Example: `https://www.autotrack.nl/aanbod/merk/toyota?data.prijs.max.0.name=20000`. Filters in the URL are kept; results come newest first unless the URL sorts otherwise. AutoTrack is testing a new site for some visitors: URLs with `pricefrom=` or `sort=age` come from that test and are not supported. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many cars (newest first). Example: `100`. Default: `100`. |
| `includeDetails` | boolean | Also open each car's page: dealer description, full equipment list by category, dealer street address, postcode and coordinates, seats, gears, drive, APK (MOT) expiry, fuel consumption, CO2, emission class, top speed, acceleration, warranties (BOVAG ...), first owner, maintenance booklet. Adds $1.00 per 1,000 cars on top of the base price. One extra request per car (about 10 s per 100 cars). Default: `false`. |
| `trackHistory` | boolean | Keep the price and status of every car in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 cars that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `golf-utrecht`. Default: `"cars-history"`. |

## Examples

**Newest Volkswagen Golf listings on AutoTrack**: Scrape the newest used Volkswagen Golf cars on AutoTrack (autotrack.nl) up to EUR 20,000: price, mileage, year, fuel, gearbox, power, licence plate, photos and dealer phone. Export to JSON, CSV or Excel.

```json
{
  "make": "Volkswagen",
  "model": "Golf",
  "condition": "used",
  "max_price": 20000,
  "limit": 200
}
```

**Electric cars near Utrecht with price history on AutoTrack**: Scrape electric cars for sale within 25 km of Utrecht from AutoTrack (autotrack.nl) and track their prices over time: price drops, mileage, battery, dealer and full details. Run it daily to catch price cuts.

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
| `id` | string | Stable listing ID, `autotrack:<advertentieId>` |
| `source` | string | Short site name (autotrack) |
| `url` | string | Car page on autotrack.nl |
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
| `vin` | string | Always null: AutoTrack shows the licence plate, not the VIN |
| `seller_type` | string | dealer (AutoTrack lists dealer cars only) |
| `seller_name` | string | Dealer name |
| `seller_phone` | string | Dealer phone number |
| `seller_url` | string | Dealer's stock page on AutoTrack |
| `address` | string | Details add-on: dealer street, postcode and town |
| `city` | string | Dealer's town |
| `postcode` | string | Details add-on: dealer postcode, e.g. 4157JA |
| `region` | string | Dealer's province, e.g. Gelderland |
| `country` | string | NL |
| `lat` | number | Details add-on: dealer latitude |
| `lon` | number | Details add-on: dealer longitude |
| `listed_at` | string | When the car was listed, ISO 8601 UTC |
| `image_urls` | array | All photo URLs |
| `description` | string | Details add-on: the dealer's description (Dutch) |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `status` | string | available, reserved, sold or expected |
| `license_plate` | string | Dutch licence plate (kenteken), e.g. ST798H |
| `commercial_variant` | string | The dealer's own variant line |
| `vehicle_type` | string | personenauto, bedrijfswagen or overig |
| `fuel_secondary` | string | Second fuel (electric for hybrids, lpg ...) |
| `price_excl_vat` | number | Price without VAT (commercial vehicles) |
| `vat_deductible` | boolean | VAT deductible for business buyers |
| `catalog_price` | number | New-car list price (catalogusprijs) in EUR |
| `energy_label` | string | Energy label A-G |
| `battery_kwh` | number | Battery capacity in kWh (plug-in hybrids, EVs) |
| `mileage_verified` | boolean | NAP: RDW judged the odometer history logical |
| `imported` | boolean | Imported car |
| `fully_maintained` | boolean | Dealer states 100% maintenance history |
| `road_tax_min` | number | Road tax per quarter, lowest province (EUR) |
| `road_tax_max` | number | Road tax per quarter, highest province (EUR) |
| `seller_id` | integer | AutoTrack dealer id |
| `seller_kind` | string | bovag_autobedrijf, officiele_merkdealer, universeel_autobedrijf ... |
| `seller_whatsapp` | string | Dealer WhatsApp number |
| `rdw_report_url` | string | RDW vehicle report download link |
| `autoweek_report_url` | string | Autoweek report PDF |
| `video_urls` | array | Dealer video URLs |
| `cross_reference_id` | string | Listing uuid shared with AutoScout24 |
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
| `first_owner` | boolean | Details add-on: first owner |
| `maintenance_booklet` | boolean | Details add-on: maintenance booklet present |
| `warranties` | array | Details add-on: warranties, e.g. BOVAG Garantie |
| `updated_at` | string | Details add-on: when the dealer last changed the listing |
| `first_seen_at` | string | Price history add-on: when this car was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: status changed since the previous run |
| `price_history` | array | Price history add-on: `[{at, price, status}]`, last 20 |

## Price

Pay per result: $0.50 per 1,000 car listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One car enriched with its AutoTrack page: dealer description, full equipment list, dealer street address, postcode and coordinates, seats, gears, drive, APK expiry, consumption, CO2, emission class, performance, warranties, first owner, maintenance booklet.
- Price history: $1.00 per 1,000. One car that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a car is free.

Store page, reviews and full documentation: [AutoTrack Car Listings Scraper](https://apify.com/carabain-digital/autotrack-listings)

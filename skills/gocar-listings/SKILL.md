---
name: gocar-listings
description: "Scrape car listings from Gocar.be, the Belgian car marketplace (NL/FR): used, new, leasing, vans and classic cars. Filter by make, model, year, price, mileage, fuel, gearbox, body, seller and postcode radius. Newest first. Add-ons: full details with Car-Pass link, price history. Runs the Apify Actor carabain-digital/gocar-listings. Use for tasks like: Used Volkswagen Golf for sale in Belgium on Gocar; Electric cars near Gent with price history on Gocar."
---

# Gocar.be Car Listings Scraper

Scrape car listings from Gocar.be, the Belgian car marketplace (NL/FR): used, new, leasing, vans and classic cars. Filter by make, model, year, price, mileage, fuel, gearbox, body, seller and postcode radius. Newest first. Add-ons: full details with Car-Pass link, price history.

Runs on Apify as the Actor [`carabain-digital/gocar-listings`](https://apify.com/carabain-digital/gocar-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 10,000 car listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~gocar-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~gocar-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"make": "Volkswagen", "model": "Golf", "condition": "used", "limit": 200}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/gocar-listings").call(run_input={"make": "Volkswagen", "model": "Golf", "condition": "used", "limit": 200})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/gocar-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `make` | string | Car make. Spelling is forgiving (matched against Gocar's make list): `Volkswagen`, `BMW`, `mercedes-benz`, `Land Rover`. Leave empty for all makes. |
| `model` | string | Model of that make as Gocar names it (Dutch names): `Golf`, `3 Reeks` (or `3-reeks`), `C-Klasse`, `Model Y`. Needs a make. An unknown model fails with the list of known ones. |
| `min_year` | integer | Earliest year of first registration. Example: `2018`. New cars without a registration are left out when set. |
| `max_year` | integer | Latest year of first registration. Example: `2022`. |
| `min_price` | integer | Lowest price in euro. Example: `5000`. |
| `max_price` | integer | Highest price in euro. Example: `25000`. |
| `max_mileage` | integer | Highest mileage in km. Example: `100000`. |
| `fuel_type` | string | Fuel type. Example: `electric`. Gocar does not separate plug-in hybrids, so `plugin_hybrid` returns all hybrids. Values: `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`, `lpg`, `cng`, `hydrogen`, `ethanol`, `other`. |
| `transmission` | string | Gearbox. Example: `automatic`. Values: `automatic`, `manual`. |
| `body_type` | string | Body type. Example: `suv`. Values: `hatchback`, `sedan`, `wagon`, `suv`, `coupe`, `convertible`, `minivan`, `van`, `pickup`, `other`. |
| `condition` | string | New or used cars. Example: `used`. Values: `any`, `used`, `new`. Default: `"any"`. |
| `seller_type` | string | Dealer or private seller. Example: `dealer`. Gocar is ~99% dealers. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `location` | string | Search around a Belgian postcode or city (Dutch or French name). Example: `9000`, `Gent`, `Liège`. Radius searches only find dealers: Gocar has no coordinates for private sellers. |
| `radius_km` | integer | Distance around the location, in km. Example: `25`. Only used with a location. Default: `25`. |
| `start_urls` | array | Search result pages copied from gocar.be (NL or FR); they replace the filters above. Example: `https://gocar.be/nl/search?gbrands=Volkswagen&gmodels=Golf&price=5000:15000`, `https://gocar.be/fr/search/u_electrique` or a brand/model page like `https://gocar.be/nl/autos/bmw/3-reeks`. All filters in the URL are kept; results always come newest first (the sort and page are ignored). `limit` counts across all URLs. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `500`. All ~55,000 Gocar listings are reachable. Default: `100`. |
| `includeDetails` | boolean | Also open each listing page: all photos, full description, engine size, cylinders, gears, interior colour and upholstery, inspection date, exact registration date, the Car-Pass mileage history link, dealer showroom page, website and phone. Adds $1.00 per 1,000 listings. About 1 s extra per 10 listings. Default: `false`. |
| `trackHistory` | boolean | Keep the price of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct and price_history. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Example: `golf-watch`. Default: `"cars-history"`. |

## Examples

**Used Volkswagen Golf for sale in Belgium on Gocar**: Scrape the newest used Volkswagen Golf listings on Gocar.be: price, mileage, year, fuel, gearbox, equipment, dealer and city. Export to JSON, CSV or Excel or schedule it to catch new cars.

```json
{
  "make": "Volkswagen",
  "model": "Golf",
  "condition": "used",
  "limit": 200
}
```

**Electric cars near Gent with price history on Gocar**: Track the price of used electric cars sold by dealers within 30 km of Gent on Gocar.be: price drops, mileage, battery, make and model. Run it daily to catch price cuts.

```json
{
  "fuel_type": "electric",
  "condition": "used",
  "location": "Gent",
  "radius_km": 30,
  "limit": 300,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `title`, `year`, `mileage`, `fuel_type`, `price`, `city`, `seller_type`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `gocar:<listing id>` |
| `source` | string | Short site name (gocar) |
| `url` | string | Listing page on gocar.be |
| `title` | string | Make, model and version as the seller wrote them |
| `make` | string | Car make, e.g. Volkswagen |
| `model` | string | Model, e.g. Tiguan |
| `variant` | string | Version / trim as the seller wrote it |
| `condition` | string | used, new or classic |
| `price` | number | Price in EUR (for leasing offers the car's price; see lease_price_monthly). Null when on request |
| `price_currency` | string | EUR |
| `price_type` | string | fixed, auction (bid on it; price = reference price) or on_request |
| `mileage` | integer | Odometer reading |
| `mileage_unit` | string | km |
| `year` | integer | Year of first registration |
| `first_registration` | string | First registration, YYYY-MM |
| `fuel_type` | string | petrol, diesel, electric, hybrid (incl. plug-in), lpg, cng, hydrogen, ethanol or other |
| `transmission` | string | manual or automatic |
| `body_type` | string | sedan, hatchback, wagon, suv, coupe, convertible, van, pickup, minivan or other |
| `power_kw` | integer | Engine power in kW |
| `power_hp` | integer | Engine power in hp (PK/CH) |
| `engine_cc` | integer | Details add-on: engine displacement in cc |
| `color` | string | Exterior colour as the site shows it (Dutch), e.g. Blauw |
| `doors` | integer | Number of doors |
| `seats` | integer | Number of seats |
| `vin` | string | VIN, only when the seller wrote it in the description (Gocar has no VIN field) |
| `seller_type` | string | dealer or private |
| `seller_name` | string | Dealer name (null for private sellers) |
| `seller_phone` | string | Details add-on: Gocar's call-routing number with the listing extension, e.g. +3223353040,5367998 (only for dealers that use it) |
| `seller_url` | string | Details add-on: dealer showroom page on gocar.be |
| `seller_website` | string | Details add-on: dealer's own website |
| `address` | string | Dealer street address, postcode and city |
| `city` | string | City of the seller |
| `postcode` | string | Postcode of the seller |
| `region` | string | Always null (Gocar has no region field) |
| `country` | string | ISO country code (BE) |
| `lat` | number | Latitude of the dealer (null for private sellers) |
| `lon` | number | Longitude of the dealer (null for private sellers) |
| `listed_at` | string | When the listing was published, UTC ISO 8601 |
| `image_urls` | array | Photo URLs: the cover photo; all photos with the details add-on |
| `description` | string | Seller's description (search results carry up to ~800 characters; the details add-on gives the full text with line breaks) |
| `fuel_type_original` | string | Fuel as the site shows it, e.g. Hybride - Benzine |
| `body_type_original` | string | Body style as the site shows it, e.g. Break/Stationwagon |
| `drive` | string | front, rear or awd when the seller filled it in |
| `emission_class` | string | Euro emission class, e.g. Euro 6d |
| `co2_g_km` | integer | CO2 emissions in g/km |
| `battery_capacity_kwh` | number | Battery capacity of electric cars in kWh |
| `equipment` | array | Equipment / options list (Dutch labels), e.g. Apple CarPlay, Panoramisch dak |
| `warranty_months` | integer | Warranty in months |
| `vat_deductible` | boolean | True when VAT is deductible (company buyers) |
| `is_leasing` | boolean | True for leasing offers |
| `lease_price_monthly` | number | Monthly lease price in EUR for leasing offers |
| `has_carpass` | boolean | True when a Car-Pass (Belgian mileage history) is available |
| `carpass_url` | string | Details add-on: link to the public Car-Pass mileage history report |
| `damaged` | boolean | True when the car is listed as damaged |
| `vehicle_type` | string | car, van or classic |
| `images_count` | integer | Number of photos |
| `seller_id` | integer | Gocar dealer (point of sale) ID |
| `registration_date` | string | Details add-on: exact first registration date, YYYY-MM-DD |
| `cylinders` | integer | Details add-on: number of cylinders |
| `gears` | integer | Details add-on: number of gears |
| `interior_color` | string | Details add-on: interior colour |
| `upholstery` | string | Details add-on: upholstery, e.g. Leder |
| `inspection_valid_until` | string | Details add-on: technical inspection valid until, as the site writes it |
| `specs` | object | Details add-on: the full spec table as site label -> value (null = not filled in) |
| `details_fetched` | boolean | Details add-on: true when the listing page was fetched |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: always false (Gocar has no status) |
| `price_history` | array | Price history add-on: price points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $0.50 per 1,000 car listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One listing enriched with its listing page: all photos, full description, engine size, cylinders, gears, interior, inspection date, exact registration date, Car-Pass mileage history link, dealer showroom page, website and phone.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [Gocar.be Car Listings Scraper](https://apify.com/carabain-digital/gocar-listings?fpr=4ijnom)

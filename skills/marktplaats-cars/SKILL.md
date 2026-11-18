---
name: marktplaats-cars
description: "Scrape car listings from Marktplaats (marktplaats.nl), the biggest classifieds site in the Netherlands: make, model, price, mileage, year, fuel, gearbox, body, seller and photos. Filter by make, model, price, year, mileage, postcode. Add-ons: full details with dealer phone, price history. Runs the Apify Actor carabain-digital/marktplaats-cars. Use for tasks like: Volkswagen Golf for sale on Marktplaats: newest listings; Dealer cars near Utrecht with price history on Marktplaats."
---

# Marktplaats Car Listings Scraper

Scrape car listings from Marktplaats (marktplaats.nl), the biggest classifieds site in the Netherlands: make, model, price, mileage, year, fuel, gearbox, body, seller and photos. Filter by make, model, price, year, mileage, postcode. Add-ons: full details with dealer phone, price history.

Runs on Apify as the Actor [`carabain-digital/marktplaats-cars`](https://apify.com/carabain-digital/marktplaats-cars). The user needs an Apify account and an API token ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)); runs are billed to their account.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~marktplaats-cars/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~marktplaats-cars/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"make": "Volkswagen", "model": "Golf", "min_year": 2015, "max_price": 20000, "limit": 200}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/marktplaats-cars").call(run_input={"make": "Volkswagen", "model": "Golf", "min_year": 2015, "max_price": 20000, "limit": 200})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/marktplaats-cars`.

## Input

| Field | Type | Description |
|---|---|---|
| `make` | string | Car make as Marktplaats lists it. Example: `Volkswagen`, `BMW`, `Mercedes-Benz` (also `VW`, `Mercedes`); `Bestelauto's` for vans. Leave empty for all makes. |
| `model` | string | Model within the make (needs `make`); an unknown model fails with the list of the site's models. Example: `Golf`, `3-Serie`, `A4`. |
| `min_year` | integer | Oldest construction year. Example: `2015`. |
| `max_year` | integer | Newest construction year. Example: `2022`. |
| `min_price` | integer | Lowest asking price in euro. Example: `5000`. |
| `max_price` | integer | Highest asking price in euro. Example: `20000`. |
| `max_mileage` | integer | Highest odometer reading in km. Example: `150000`. |
| `fuel_type` | string | Fuel type. Example: `diesel`. Values: `any`, `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`, `lpg`, `cng`, `hydrogen`, `other`. Default: `"any"`. |
| `transmission` | string | Gearbox. Example: `automatic`. Values: `any`, `manual`, `automatic`. Default: `"any"`. |
| `body_type` | string | Body type. Example: `suv`. Vans are a make-like section on the site: use `make` for them. Values: `any`, `hatchback`, `sedan`, `wagon`, `suv`, `coupe`, `convertible`, `minivan`, `other`. Default: `"any"`. |
| `condition` | string | New or used cars. Example: `used`. Values: `any`, `used`, `new`. Default: `"any"`. |
| `seller_type` | string | Dealers or private sellers. Example: `private`. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `location` | string | Dutch postcode (4 digits, optionally with the 2 letters). Example: `3511AB` or `1012`. The site searches by postcode + distance only, so city names are rejected. |
| `radius_km` | integer | Distance around the postcode. Example: `25`. Only used with a postcode. Default: `25`. |
| `start_urls` | array | Car search pages copied from marktplaats.nl; they replace the filters above. Example: `https://www.marktplaats.nl/l/auto-s/audi/f/diesel/474/#PriceCentsTo:2000000\|constructionYearFrom:2015`. Filters in the path and after the `#` are kept; results always come newest first. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `100`. The site serves about 2,000 listings per search: narrow the filters (or add more search URLs) for more. Default: `100`. |
| `includeDetails` | boolean | Also open each listing's page: full description, all photos, trim/version, licence plate, APK date, previous owners, colour, doors, seats, engine, gears, drive, options list, seller type, dealer phone, website and postcode, exact publication date, views and favourites. Adds $1.00 per 1,000 listings on top of the base price. One extra request per listing (about 6 s per 100 listings). Default: `false`. |
| `trackHistory` | boolean | Keep the price of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `golf-watch`. Default: `"cars-history"`. |

## Examples

**Volkswagen Golf for sale on Marktplaats: newest listings**: Scrape the newest Volkswagen Golf listings from Marktplaats (marktplaats.nl): price, mileage, year, fuel, gearbox, city and seller. Export to JSON, CSV or Excel, or schedule it to catch new cars.

```json
{
  "make": "Volkswagen",
  "model": "Golf",
  "min_year": 2015,
  "max_price": 20000,
  "limit": 200
}
```

**Dealer cars near Utrecht with price history on Marktplaats**: Scrape dealer car listings within 25 km of Utrecht from Marktplaats and track their price over time: asking price, price drops, mileage, year, dealer name and phone. Run it daily to catch price cuts.

```json
{
  "location": "3511",
  "radius_km": 25,
  "seller_type": "dealer",
  "limit": 300,
  "includeDetails": true,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `title`, `price`, `year`, `mileage`, `fuel_type`, `city`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `<source>:<site id>` |
| `source` | string | Short site name |
| `url` | string | Public listing URL |
| `title` | string | Listing title as the seller wrote it |
| `make` | string | Make, e.g. Volkswagen (null for the site's vans/trucks/oldtimer sections when the seller set none) |
| `model` | string | Model as the site names it, e.g. Golf, 3-Serie |
| `variant` | string | Details add-on: trim/version, e.g. 1.5 TSI R-Line |
| `condition` | string | new, used or classic (oldtimer section); null when the seller did not say |
| `price` | number | Asking price in EUR; for bid listings the minimum bid. Null when the price is on request |
| `price_currency` | string | EUR |
| `price_type` | string | fixed, negotiable (bids accepted) or on_request |
| `mileage` | integer | Odometer reading |
| `mileage_unit` | string | km |
| `year` | integer | Construction year (bouwjaar) |
| `first_registration` | string | YYYY (the construction year; the site does not give a registration month) |
| `fuel_type` | string | petrol, diesel, electric, hybrid, lpg, cng, hydrogen or other |
| `transmission` | string | manual or automatic |
| `body_type` | string | sedan, hatchback, wagon, suv, coupe, convertible, van, minivan or other |
| `power_kw` | integer | Engine power in kW |
| `power_hp` | integer | Engine power in hp |
| `engine_cc` | integer | Engine displacement in cc |
| `color` | string | Exterior colour (Dutch, as on the site) |
| `doors` | integer | Number of doors |
| `seats` | integer | Number of seats |
| `vin` | string | Details add-on: VIN when the seller put one in the description |
| `seller_type` | string | dealer or private (always set with the details add-on or a seller_type filter) |
| `seller_name` | string | Seller or dealer name |
| `seller_phone` | string | Details add-on: phone number (dealers mostly) |
| `seller_url` | string | Details add-on: seller profile page |
| `address` | string | Postcode and city |
| `city` | string | City |
| `postcode` | string | Details add-on: postcode (dealers) |
| `region` | string | Not given by the site |
| `country` | string | Country code |
| `lat` | number | Latitude |
| `lon` | number | Longitude |
| `listed_at` | string | Search: date of the listing or of its latest paid bump (YYYY-MM-DD); details add-on: exact first publication timestamp |
| `image_urls` | array | Photo URLs (search: up to 4; details add-on: all) |
| `description` | string | Search: first ~300 characters; details add-on: full text |
| `seller_id` | integer | Seller account id |
| `promoted` | boolean | Paid 'Dagtopper' ad (bumped to the top daily) |
| `reserved` | boolean | Marked as reserved |
| `energy_label` | string | Energy label (NL) |
| `emission_class` | string | Euro norm (BE) |
| `car_pass_url` | string | Public Car-Pass mileage certificate link (2dehands) |
| `equipment` | array | Options list, e.g. Cruise Control, Navigatiesysteem |
| `fuel_type_original` | string | Fuel as the site writes it |
| `body_type_original` | string | Body type as the site writes it |
| `category` | string | The site's car section (make, or Bestelauto's/Vrachtwagens/Oldtimers) |
| `license_plate` | string | Details add-on: licence plate (NL) |
| `apk_until` | string | Details add-on: APK (MOT) expiry date (NL) |
| `previous_owners` | integer | Details add-on: number of previous owners (NL) |
| `imported` | boolean | Details add-on: imported car (NL) |
| `gears` | integer | Details add-on: number of gears |
| `drive` | string | Details add-on: driven wheels |
| `seller_website` | string | Details add-on: dealer website |
| `seller_active_years` | integer | Details add-on: years the seller has been on the site |
| `cylinders` | integer | Details add-on: number of cylinders |
| `interior_color` | string | Details add-on: interior colour |
| `upholstery` | string | Details add-on: upholstery, e.g. Leder |
| `service_history` | string | Details add-on: maintenance history, e.g. Dealer onderhouden |
| `new_price_text` | string | Details add-on: list price when new, as the site shows it (NL) |
| `co2_g_km` | integer | Details add-on: CO2 emission in g/km |
| `empty_weight_kg` | integer | Details add-on: empty weight in kg |
| `nap_status` | boolean | Details add-on: NAP odometer check available (NL) |
| `view_count` | integer | Details add-on: page views |
| `favorite_count` | integer | Details add-on: times saved as favourite |
| `details_fetched` | boolean | True when the details add-on enriched this item |
| `first_seen_at` | string | Price-history add-on: first time this listing was seen |
| `previous_price` | number | Price-history add-on: price at the previous run |
| `price_change` | number | Price-history add-on: change since the previous run |
| `price_change_pct` | number | Price-history add-on: change in percent |
| `status_changed` | boolean | Price-history add-on: status changed since the previous run |
| `price_history` | array | Price-history add-on: last 20 price points |
| `scraped_at` | string | When it was scraped (UTC) |

## Price

Pay per result: $0.80 per 1,000 car listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One listing enriched with its page: full description, all photos, trim, licence plate, APK date, previous owners, colour, doors, seats, engine, gears, options, seller type, dealer phone, website and postcode, exact publication date, views and favourites.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [Marktplaats Car Listings Scraper](https://apify.com/carabain-digital/marktplaats-cars)

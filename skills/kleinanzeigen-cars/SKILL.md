---
name: kleinanzeigen-cars
description: "Scrape car listings from Kleinanzeigen (kleinanzeigen.de), Germany's biggest classifieds site: make, model, price, mileage, year, fuel, gearbox, body, seller and photos. Filter by make, model, price, year, mileage, location. Add-ons: full details with dealer phone, price history. Runs the Apify Actor carabain-digital/kleinanzeigen-cars. Use for tasks like: Volkswagen Golf for sale on Kleinanzeigen: newest listings; Dealer cars near Hamburg with price history on Kleinanzeigen."
---

# Kleinanzeigen Car Listings Scraper

Scrape car listings from Kleinanzeigen (kleinanzeigen.de), Germany's biggest classifieds site: make, model, price, mileage, year, fuel, gearbox, body, seller and photos. Filter by make, model, price, year, mileage, location. Add-ons: full details with dealer phone, price history.

Runs on Apify as the Actor [`carabain-digital/kleinanzeigen-cars`](https://apify.com/carabain-digital/kleinanzeigen-cars?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 10,000 car listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~kleinanzeigen-cars/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~kleinanzeigen-cars/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"make": "Volkswagen", "model": "Golf", "min_year": 2015, "max_price": 20000, "limit": 200}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/kleinanzeigen-cars").call(run_input={"make": "Volkswagen", "model": "Golf", "min_year": 2015, "max_price": 20000, "limit": 200})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/kleinanzeigen-cars`.

## Input

| Field | Type | Description |
|---|---|---|
| `make` | string | Car make as Kleinanzeigen lists it. Example: `Volkswagen`, `BMW`, `Mercedes-Benz` (also `VW`, `Mercedes`). Leave empty for all makes. An unknown make fails with the site's list. |
| `model` | string | Model within the make (needs `make`); an unknown model fails with the list of the site's models. Example: `Golf`, `3er`, `A4`. |
| `min_year` | integer | Oldest construction year. Example: `2015`. |
| `max_year` | integer | Newest construction year. Example: `2022`. |
| `min_price` | integer | Lowest asking price in euro. Example: `5000`. |
| `max_price` | integer | Highest asking price in euro. Example: `20000`. |
| `max_mileage` | integer | Highest odometer reading in km. Example: `150000`. |
| `fuel_type` | string | Fuel type. Example: `diesel`. Values: `any`, `petrol`, `diesel`, `electric`, `hybrid`, `lpg`, `cng`, `other`. Default: `"any"`. |
| `transmission` | string | Gearbox. Example: `automatic`. Values: `any`, `manual`, `automatic`. Default: `"any"`. |
| `body_type` | string | Body type. Example: `suv`. `hatchback` is the site's Kleinwagen, `minivan` its Bus/Van. Values: `any`, `hatchback`, `sedan`, `wagon`, `suv`, `coupe`, `convertible`, `minivan`, `other`. Default: `"any"`. |
| `seller_type` | string | Dealers or private sellers. Example: `private`. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `location` | string | German postcode or city. Example: `10115` or `Hamburg`. The site resolves it to its first match; combine with a radius. |
| `radius_km` | integer | Distance around the location. Example: `25`. Used with a location; without one the site's default applies. |
| `start_urls` | array | Car search pages copied from kleinanzeigen.de; they replace the filters above. Example: `https://www.kleinanzeigen.de/s-autos/volkswagen/c216+autos.marke_s:volkswagen+autos.ez_i:2015%2C`. Results come newest first; wanted ads are skipped unless the URL has `anzeige:gesuche`. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `100`. One search serves at most 50 pages (about 1,300 listings): narrow the filters (or add more search URLs) for more. Default: `100`. |
| `includeDetails` | boolean | Also open each listing's page: make and model, fuel, gearbox, power, body type, colour, doors, HU date, emission class, equipment list, full description, all photos, seller type, name and ID, dealer phone, exact postcode, coordinates and publication date. Adds $1.00 per 1,000 listings on top of the base price. One extra request per listing (about 12 s per 100 listings). Default: `false`. |
| `trackHistory` | boolean | Keep the price of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `golf-watch`. Default: `"cars-history"`. |

## Examples

**Volkswagen Golf for sale on Kleinanzeigen: newest listings**: Scrape the newest Volkswagen Golf listings from Kleinanzeigen (kleinanzeigen.de): price, mileage, year, city and photos. Export to JSON, CSV or Excel, or schedule it to catch new cars.

```json
{
  "make": "Volkswagen",
  "model": "Golf",
  "min_year": 2015,
  "max_price": 20000,
  "limit": 200
}
```

**Dealer cars near Hamburg with price history on Kleinanzeigen**: Scrape dealer car listings within 25 km of Hamburg from Kleinanzeigen and track their price over time: asking price, price drops, mileage, year, dealer name and phone. Run it daily to catch price cuts.

```json
{
  "location": "Hamburg",
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
| `first_registration` | string | YYYY-MM (the construction year and month) |
| `fuel_type` | string | petrol, diesel, electric, hybrid, lpg, cng, hydrogen or other |
| `transmission` | string | manual or automatic |
| `body_type` | string | sedan, hatchback, wagon, suv, coupe, convertible, van, minivan or other |
| `power_kw` | integer | Engine power in kW |
| `power_hp` | integer | Engine power in hp |
| `engine_cc` | integer | Engine displacement in cc |
| `color` | string | Exterior colour (German, as on the site; details add-on) |
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
| `promoted` | boolean | Paid top ad (no date on the card, bumped to the top) |
| `reserved` | boolean | Marked as reserved |
| `energy_label` | string | Energy label (NL) |
| `emission_class` | string | Euro norm (BE) |
| `car_pass_url` | string | Not used on this site (always null) |
| `equipment` | array | Options list, e.g. Cruise Control, Navigatiesysteem |
| `fuel_type_original` | string | Fuel as the site writes it |
| `body_type_original` | string | Body type as the site writes it |
| `category` | string | Details add-on: damage status as the site writes it ('Unbeschädigtes Fahrzeug', 'Beschädigtes Fahrzeug') |
| `license_plate` | string | Details add-on: licence plate (NL) |
| `apk_until` | string | Details add-on: HU (TUeV) expiry as the site writes it, e.g. 'Mai 2028' |
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

Pay per result: $0.50 per 1,000 car listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One listing enriched with its page: make and model, fuel, gearbox, power, body type, colour, doors, HU date, emission class, equipment, full description, all photos, seller type, name and ID, dealer phone, exact postcode, coordinates and publication date.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [Kleinanzeigen Car Listings Scraper](https://apify.com/carabain-digital/kleinanzeigen-cars?fpr=4ijnom)

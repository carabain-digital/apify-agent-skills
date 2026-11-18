---
name: gaspedaal-listings
description: "Scrape used-car listings from Gaspedaal.nl, the Dutch car aggregator: make, model, price, mileage, year, fuel, gearbox, power, city and the source site (AutoTrack, AutoScout24, dealer). Filter by make, price, year, km, postcode. Newest first. Add-ons: dealer details, price history. Runs the Apify Actor carabain-digital/gaspedaal-listings. Use for tasks like: Volkswagen Golf under €20,000: newest on Gaspedaal; Used electric cars near Utrecht with price history on Gaspedaal."
---

# Gaspedaal Car Listings Scraper

Scrape used-car listings from Gaspedaal.nl, the Dutch car aggregator: make, model, price, mileage, year, fuel, gearbox, power, city and the source site (AutoTrack, AutoScout24, dealer). Filter by make, price, year, km, postcode. Newest first. Add-ons: dealer details, price history.

Runs on Apify as the Actor [`carabain-digital/gaspedaal-listings`](https://apify.com/carabain-digital/gaspedaal-listings), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 3,571 car listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~gaspedaal-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~gaspedaal-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"make": "Volkswagen", "model": "Golf", "min_year": 2018, "max_price": 20000, "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/gaspedaal-listings").call(run_input={"make": "Volkswagen", "model": "Golf", "min_year": 2018, "max_price": 20000, "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/gaspedaal-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `make` | string | Car make as Gaspedaal shows it. Example: `Volkswagen`, `BMW`, `Mercedes-Benz`, `Land Rover`. Leave empty for all makes. An unknown make fails the run with a clear message instead of returning every car. Ignored when you give search URLs. |
| `model` | string | Model of that make as Gaspedaal shows it. Example: `Golf`, `3-Serie`, `C-Klasse`, `Model 3`. Needs a make. |
| `min_year` | integer | Oldest construction year. Example: `2018`. |
| `max_year` | integer | Newest construction year. Example: `2023`. |
| `min_price` | integer | Lowest asking price in euro incl. VAT. Example: `5000`. |
| `max_price` | integer | Highest asking price in euro incl. VAT. Example: `25000`. |
| `max_mileage` | integer | Highest odometer reading in km. Example: `100000`. |
| `fuel_type` | string | Fuel type. Gaspedaal has one hybrid category, so `plugin_hybrid` returns all hybrids. Example: `electric`. Values: `any`, `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`, `lpg`, `cng`, `hydrogen`. Default: `"any"`. |
| `transmission` | string | Gearbox. Example: `automatic`. Values: `any`, `manual`, `automatic`. Default: `"any"`. |
| `body_type` | string | Body type. Example: `suv`. Values: `any`, `hatchback`, `sedan`, `wagon`, `suv`, `coupe`, `convertible`, `minivan`, `van`, `pickup`, `other`. Default: `"any"`. |
| `condition` | string | Gaspedaal lists used cars (occasions) only, including demo and nearly-new cars; `new` is not available. Example: `used`. Values: `any`, `used`. Default: `"any"`. |
| `seller_type` | string | Dealer or private seller. Example: `private`. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `location` | string | Dutch postcode to search around, 4 digits or 4 digits + 2 letters. Example: `3511` (Utrecht) or `1012AB`. City names are not supported by Gaspedaal's search. Empty = all of the Netherlands. |
| `radius_km` | integer | Distance from the postcode in km (default 25). Example: `50`. |
| `start_urls` | array | Search result pages copied from gaspedaal.nl; they replace the filters above. Example: `https://www.gaspedaal.nl/volkswagen/golf/diesel?pmax=20000&bmin=2018`. Filters in the URL are kept; results always come newest first and the page number is ignored. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many cars (newest first). Example: `100`. Default: `100`. |
| `includeDetails` | boolean | Also follow each car to its source listing (AutoScout24, AutoTrack, dealer sites, ...): the direct source URL of every site that carries it, plus the description, all photos, licence plate (kenteken), seats, first registration month, equipment, dealer name, phone, address and coordinates when the source page has them. Adds $1.00 per 1,000 cars on top of the base price, charged only for cars a source page enriched (cars only on AutoWereld get their URLs for free). About 1 s per 10 cars. Default: `false`. |
| `trackHistory` | boolean | Keep the price of every car in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 cars that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Example: `golf-diesel-history`. Default: `"cars-history"`. |

## Examples

**Volkswagen Golf under €20,000: newest on Gaspedaal**: Scrape the newest used Volkswagen Golf listings under €20,000 from 2018 or later on Gaspedaal.nl, the Dutch car aggregator: price, mileage, year, fuel, gearbox, power, city and the source site. Export to JSON, CSV or Excel.

```json
{
  "make": "Volkswagen",
  "model": "Golf",
  "min_year": 2018,
  "max_price": 20000,
  "limit": 100
}
```

**Used electric cars near Utrecht with price history on Gaspedaal**: Scrape used electric cars within 50 km of Utrecht from Gaspedaal.nl and track their price over time: price drops, mileage, year, power and dealer details. Run it daily to catch price drops on EVs.

```json
{
  "fuel_type": "electric",
  "location": "3511",
  "radius_km": 50,
  "limit": 200,
  "includeDetails": true,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `title`, `year`, `mileage`, `price`, `fuel_type`, `city`, `source_site`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `gaspedaal:<advertentieId>` |
| `source` | string | Short site name (gaspedaal) |
| `url` | string | Gaspedaal link to the listing: a click-out that opens the car on the source site (AutoTrack, AutoScout24, a dealer site ...) in a browser |
| `title` | string | Make, model and variant |
| `make` | string | Make, e.g. Volkswagen |
| `model` | string | Model, e.g. Golf |
| `variant` | string | Variant / trim as the seller wrote it, e.g. 1.5 eTSI Style |
| `condition` | string | used (Gaspedaal lists occasions) |
| `price` | number | Asking price in EUR incl. VAT |
| `price_currency` | string | EUR |
| `price_type` | string | fixed, or on_request when no price is given |
| `mileage` | number | Odometer reading |
| `mileage_unit` | string | km |
| `year` | integer | Construction year (bouwjaar) |
| `first_registration` | string | YYYY, or YYYY-MM with the Full details add-on when the source page has it |
| `fuel_type` | string | petrol, diesel, electric, hybrid, lpg, cng, hydrogen or other (Gaspedaal does not split plug-in hybrids from hybrids) |
| `transmission` | string | manual or automatic |
| `body_type` | string | sedan, hatchback, wagon, suv, coupe, convertible, van, pickup, minivan or other |
| `power_kw` | number | Engine power in kW |
| `power_hp` | number | Engine power in hp (from kW) |
| `engine_cc` | number | Engine displacement in cc |
| `color` | string | Body colour in English, e.g. black, grey |
| `doors` | integer | Number of doors |
| `seats` | integer | Details add-on: number of seats (when the source page has it) |
| `vin` | string | Details add-on: VIN, when a source page publishes it (rare in the Netherlands) |
| `license_plate` | string | Details add-on: Dutch licence plate (kenteken) without dashes, e.g. NDX22R, from AutoTrack or AutoScout24 |
| `seller_type` | string | dealer or private: from the seller filter, the dealer link, or the source page (Details add-on) |
| `seller_name` | string | Details add-on: dealer or seller name |
| `seller_phone` | string | Details add-on: seller phone number |
| `seller_url` | string | The dealer's page on Gaspedaal (all its cars), when Gaspedaal links it |
| `seller_website` | string | Details add-on: the dealer's page or website on the source site |
| `address` | string | Details add-on: dealer street address, postcode and city |
| `city` | string | City of the seller |
| `postcode` | string | Details add-on: seller postcode |
| `region` | string | Province, e.g. Noord-Brabant |
| `country` | string | NL |
| `lat` | number | Details add-on: latitude of the seller |
| `lon` | number | Details add-on: longitude of the seller |
| `listed_at` | string | When the car was placed on Gaspedaal, UTC ISO 8601 |
| `image_urls` | array | Photo URLs (one from the search results; all photos of the source listing with the Details add-on) |
| `description` | string | Details add-on: the seller's description text (Dutch) |
| `equipment` | array | Details add-on: equipment / options list (from AutoScout24 listings) |
| `source_site` | string | The site Gaspedaal found the car on (first non-lease source with the Details add-on), e.g. AutoTrack, AutoScout24, Dealersite, ANWB |
| `source_sites` | array | Every site that carries this car |
| `sources` | array | Every source: `{site, type (dealer\|other\|financial), url (Gaspedaal click-out), source_url (Details add-on: the real listing URL)}` |
| `source_url` | string | Details add-on: direct URL of the listing on the source site |
| `details_source` | string | Details add-on: the source site the details were read from |
| `details_fetched` | boolean | True when the Details add-on enriched this item |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: true when the status changed since the previous run |
| `price_history` | array | Price history add-on: price points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $1.40 per 1,000 car listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One car enriched from its source listing (AutoTrack, AutoScout24, dealer site): direct source URLs, description, all photos, licence plate, seats, first registration month, equipment, dealer name, phone, address and coordinates. Cars no source page enriched are not charged.
- Price history: $1.00 per 1,000. One car that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a car is free.

Store page, reviews and full documentation: [Gaspedaal Car Listings Scraper](https://apify.com/carabain-digital/gaspedaal-listings)

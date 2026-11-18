---
name: autoscout24-listings
description: "Scrape car listings from AutoScout24, Europe's biggest car marketplace: Germany, Netherlands, Belgium, Austria, Italy, France, Spain, Luxembourg. Filter by make, model, year, price, mileage, fuel. Price rating, dealer phone. Add-ons: full details with market price, price history. Runs the Apify Actor carabain-digital/autoscout24-listings. Use for tasks like: Used VW Golf under 15,000 EUR in Germany on AutoScout24; Electric cars in the Netherlands with price history."
---

# AutoScout24 Car Listings Scraper

Scrape car listings from AutoScout24, Europe's biggest car marketplace: Germany, Netherlands, Belgium, Austria, Italy, France, Spain, Luxembourg. Filter by make, model, year, price, mileage, fuel. Price rating, dealer phone. Add-ons: full details with market price, price history.

Runs on Apify as the Actor [`carabain-digital/autoscout24-listings`](https://apify.com/carabain-digital/autoscout24-listings), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 17,857 car listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~autoscout24-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~autoscout24-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"country": "de", "make": "Volkswagen", "model": "Golf", "condition": "used", "max_price": 15000, "limit": 200}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/autoscout24-listings").call(run_input={"country": "de", "make": "Volkswagen", "model": "Golf", "condition": "used", "max_price": 15000, "limit": 200})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/autoscout24-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `country` | string | Which AutoScout24 site and country to search: cars located in that country. Example: `nl` for autoscout24.nl. Ignored when you give search URLs (their own site is used). Values: `de`, `nl`, `be`, `at`, `it`, `fr`, `es`, `lu`. Default: `"de"`. |
| `make` | string | Car make as AutoScout24 writes it. Example: `Volkswagen`, `BMW`, `Mercedes-Benz`, `Land Rover`. Leave empty for all makes. |
| `model` | string | Model of that make as in the site's URLs. Example: `Golf`, `X1`, `C-Klasse` (German names also work on the other country sites). Needs a make. If a model is not found, run the search on the site and paste its URL into Search URLs. |
| `min_year` | integer | Earliest year of first registration. Example: `2018`. |
| `max_year` | integer | Latest year of first registration. Example: `2022`. |
| `min_price` | integer | Lowest asking price in euro. Example: `5000`. |
| `max_price` | integer | Highest asking price in euro. Example: `25000`. |
| `max_mileage` | integer | Highest mileage in km. Example: `100000`. |
| `fuel_type` | string | Fuel type. Example: `electric`. AutoScout24 does not separate plug-in hybrids in its search, so `plugin_hybrid` returns all hybrids; the Full details add-on marks the plug-ins. Values: `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`, `lpg`, `cng`, `hydrogen`, `ethanol`, `other`. |
| `transmission` | string | Gearbox. Example: `automatic`. Values: `automatic`, `manual`. |
| `body_type` | string | Body type. Example: `suv`. Values: `hatchback`, `sedan`, `wagon`, `suv`, `coupe`, `convertible`, `minivan`, `van`, `pickup`, `other`. |
| `condition` | string | New or used cars. Example: `used`. Values: `any`, `used`, `new`. Default: `"any"`. |
| `seller_type` | string | Dealer or private seller. Example: `private`. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `location` | string | Search around a postcode or city in the chosen country. Example: `10115`, `München`, `Utrecht`, `Gent`. |
| `radius_km` | integer | Distance around the location, in km. Example: `50`. Only used with a location. Default: `50`. |
| `start_urls` | array | Search result pages copied from any AutoScout24 site; they replace the filters above. Example: `https://www.autoscout24.de/lst/bmw/x1?fregfrom=2018&priceto=30000`. All filters and the sort order in the URL are kept (newest first when the URL has none); the page number is ignored. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `500`. One search on AutoScout24 pages through at most 4,000 cars; above that the Actor splits the search into price bands automatically. Default: `100`. |
| `includeDetails` | boolean | Also open each listing: description, all equipment, body type, colour, doors, seats, engine size, gears, previous owners, accident and service history, warranty, coordinates, listing date, views, VAT/net price and AutoScout24's market price (median and rating bands). Adds $0.95 per 1,000 listings. About 1 s extra per 10 listings. Default: `false`. |
| `trackHistory` | boolean | Keep the price of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct and price_history. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Example: `golf-watch`. Default: `"cars-history"`. |

## Examples

**Used VW Golf under 15,000 EUR in Germany on AutoScout24**: Scrape the newest used Volkswagen Golf listings under 15,000 euro from autoscout24.de: price, AutoScout24 price rating, mileage, year, fuel, gearbox, power, city and dealer phone. Export to JSON, CSV or Excel.

```json
{
  "country": "de",
  "make": "Volkswagen",
  "model": "Golf",
  "condition": "used",
  "max_price": 15000,
  "limit": 200
}
```

**Electric cars in the Netherlands with price history**: Track used electric cars on autoscout24.nl over time: asking price, price changes, AutoScout24 price rating, mileage, year, power and dealer. Run it daily to catch price drops.

```json
{
  "country": "nl",
  "fuel_type": "electric",
  "condition": "used",
  "limit": 500,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `title`, `year`, `mileage`, `fuel_type`, `price`, `price_rating`, `city`, `seller_type`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `autoscout24:<listing uuid>` |
| `source` | string | Site short name (autoscout24) |
| `url` | string | Listing page on AutoScout24 |
| `title` | string | Make, model and trim, e.g. Volkswagen Golf 1.5 TSI Life |
| `make` | string | Make, e.g. BMW |
| `model` | string | Model, e.g. X1 |
| `variant` | string | Trim / version as the seller wrote it, e.g. xDrive28i M Sport |
| `condition` | string | used, new, demo (demonstrator or pre-registered) or classic |
| `price` | integer | Asking price in EUR (gross, incl. VAT) |
| `price_currency` | string | EUR |
| `price_type` | string | fixed, negotiable (details add-on) or on_request |
| `mileage` | integer | Mileage |
| `mileage_unit` | string | km |
| `year` | integer | Year of first registration (null for new cars) |
| `first_registration` | string | First registration, YYYY-MM |
| `fuel_type` | string | petrol, diesel, electric, hybrid, plugin_hybrid (details add-on), lpg, cng, hydrogen, ethanol or other |
| `transmission` | string | manual or automatic (semi-automatic counts as automatic) |
| `body_type` | string | Details add-on: sedan, hatchback, wagon, suv, coupe, convertible, van, minivan or other |
| `power_kw` | integer | Engine power in kW |
| `power_hp` | integer | Engine power in hp (PS) |
| `engine_cc` | integer | Details add-on: engine displacement in cm³ |
| `color` | string | Details add-on: body colour as the site names it, in the site's language |
| `doors` | integer | Details add-on: number of doors |
| `seats` | integer | Details add-on: number of seats |
| `vin` | string | Vehicle identification number: AutoScout24 does not publish it, always null |
| `seller_type` | string | dealer or private |
| `seller_name` | string | Dealer company name (or contact name) |
| `seller_phone` | string | Seller phone number, international format |
| `seller_url` | string | Dealer page on AutoScout24 |
| `address` | string | Street, postcode and city on one line (private sellers: postcode and city) |
| `city` | string | City |
| `postcode` | string | Postcode |
| `region` | string | Not published by AutoScout24, always null |
| `country` | string | ISO country code of the car, e.g. DE |
| `lat` | number | Details add-on: latitude |
| `lon` | number | Details add-on: longitude |
| `listed_at` | string | Details add-on: when the listing was created or last renewed, UTC ISO 8601 |
| `image_urls` | array | Photo URLs, 1280x960 |
| `description` | string | Details add-on: the seller's description as plain text |
| `price_rating` | string | AutoScout24's price rating vs the market: very_good, good, fair, increased or high (null when not rated) |
| `seller_rating` | number | Dealer's average review stars (1-5) |
| `seller_rating_count` | integer | Number of dealer reviews |
| `seller_contact_name` | string | Dealer contact person |
| `online_purchase` | boolean | True when the car can be bought online through AutoScout24 (smyle) |
| `details_fetched` | boolean | True when the Full details add-on enriched this item |
| `seller_phones` | array | Details add-on: all seller phone numbers |
| `seller_whatsapp` | string | Details add-on: seller WhatsApp number |
| `equipment` | array | Details add-on: equipment and options, in the site's language (e.g. Klimaanlage, Navigationssystem) |
| `gears` | integer | Details add-on: number of gears |
| `cylinders` | integer | Details add-on: number of cylinders |
| `drive_train` | string | Details add-on: drive train (front, rear, 4WD) |
| `weight_kg` | integer | Details add-on: empty weight in kg |
| `previous_owners` | integer | Details add-on: number of previous owners |
| `had_accident` | boolean | Details add-on: accident damage declared |
| `full_service_history` | boolean | Details add-on: full service history (Scheckheft) |
| `warranty` | object or string | Details add-on: warranty details when offered |
| `fuel_consumption_l_100km` | number | Details add-on: combined fuel consumption, l/100 km (kWh/100 km for EVs) |
| `co2_g_km` | number | Details add-on: CO2 emissions, g/km |
| `net_price` | integer | Details add-on: price excl. VAT, for VAT-deductible cars |
| `vat_deductible` | boolean | Details add-on: VAT deductible (dealer cars) |
| `market_price_median` | integer | Details add-on: AutoScout24's market price for comparable cars (median) |
| `price_rating_bands` | array | Details add-on: price bands behind the rating, `[[min, max], ...]` from very good to high |
| `views` | integer | Details add-on: listing page views |
| `favorites` | integer | Details add-on: times saved as favourite |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: true when the status changed |
| `price_history` | array | Price history add-on: price points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $0.28 per 1,000 car listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $0.95 per 1,000. One car listing enriched with its listing page: description, equipment, body type, colour, doors, seats, engine size, gears, previous owners, accident and service history, warranty, coordinates, listing date, views and favourites, net price and AutoScout24's market price median and rating bands.
- Price history: $1.00 per 1,000. One car listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [AutoScout24 Car Listings Scraper](https://apify.com/carabain-digital/autoscout24-listings)

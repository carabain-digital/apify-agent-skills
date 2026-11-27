---
name: autouncle-listings
description: "Scrape used and new car listings from AutoUncle, the price-comparison site, in 14 countries (DE, NL, FR, ES, IT, AT, CH, UK, DK, SE, FI, PL, PT, RO). AutoUncle price rating and market value included. Add-ons: full details, price history. Runs the Apify Actor carabain-digital/autouncle-listings. Use for tasks like: Best-priced used BMW 3-Series in Germany on AutoUncle; Electric cars in the Netherlands with price history."
---

# AutoUncle Car Listings Scraper

Scrape used and new car listings from AutoUncle, the price-comparison site, in 14 countries (DE, NL, FR, ES, IT, AT, CH, UK, DK, SE, FI, PL, PT, RO). AutoUncle price rating and market value included. Add-ons: full details, price history.

Runs on Apify as the Actor [`carabain-digital/autouncle-listings`](https://apify.com/carabain-digital/autouncle-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 10,000 car listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~autouncle-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~autouncle-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"country": "de", "make": "BMW", "model": "3-Series", "condition": "used", "max_price": 30000, "min_price_rating": 4, "limit": 200}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/autouncle-listings").call(run_input={"country": "de", "make": "BMW", "model": "3-Series", "condition": "used", "max_price": 30000, "min_price_rating": 4, "limit": 200})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/autouncle-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `country` | string | Which AutoUncle site and country to search: cars on sale in that country, in its currency. Example: `nl` for autouncle.nl. Ignored when you give search URLs (their own site is used). Values: `de`, `nl`, `fr`, `es`, `it`, `at`, `ch`, `uk`, `dk`, `se`, `fi`, `pl`, `pt`, `ro`. Default: `"de"`. |
| `make` | string | Car make. Example: `Volkswagen` (AutoUncle writes `VW`), `BMW`, `Mercedes`, `Land Rover`. Leave empty for all makes. |
| `model` | string | Model of that make, or its whole series. Example: `Golf` (all Golf generations), `Golf VIII` (one generation), `X1`, `3er` or `3-Series`, `C-Klasse`. Needs a make. A model AutoUncle does not know stops the run with examples of the names it uses. |
| `min_year` | integer | Earliest year of first registration. Example: `2018`. |
| `max_year` | integer | Latest year of first registration. Example: `2022`. |
| `min_price` | integer | Lowest asking price in the country's currency. Example: `5000`. |
| `max_price` | integer | Highest asking price in the country's currency. Example: `25000`. |
| `max_mileage` | integer | Highest mileage (km; miles on autouncle.co.uk). Example: `100000`. |
| `fuel_type` | string | Fuel type. Example: `electric`. `hybrid` is every hybrid (electric with petrol, LPG or CNG); AutoUncle has no separate plug-in filter, so `plugin_hybrid` returns the electric hybrids. Values: `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`, `lpg`, `cng`, `hydrogen`, `ethanol`, `other`. |
| `transmission` | string | Gearbox. Example: `automatic`. Values: `automatic`, `manual`. |
| `body_type` | string | Body type. Example: `suv`. Values: `hatchback`, `sedan`, `wagon`, `suv`, `coupe`, `convertible`, `minivan`, `pickup`. |
| `condition` | string | New or used cars. Example: `used`. Values: `any`, `used`, `new`. Default: `"any"`. |
| `seller_type` | string | Dealer or private seller. Example: `private`. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `min_price_rating` | integer | Only cars AutoUncle rates at least this many stars (5 = Superpreis, 4 = Guter Preis, 3 = Fairer Preis, 2 = Etwas zu teuer, 1 = Teuer). Example: `4` for deals. Leave empty for all cars, rated or not. |
| `max_days_listed` | integer | Only cars advertised for at most this many days. Example: `7` for the last week. |
| `location` | string | Search around a postcode or place in the chosen country. Example: `10115`, `München`, `Utrecht`, `SW1A 1AA`. |
| `radius_km` | integer | Distance around the location, in km. Example: `50`. Only used with a location. Default: `50`. |
| `start_urls` | array | Search result pages copied from any AutoUncle site; they replace the filters above. Example: `https://www.autouncle.de/de/gebrauchtwagen/BMW/X1?s[min_year]=2018`. All filters and the sort order in the URL are kept (newest first when the URL has none); the page number is ignored. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `500`. AutoUncle pages through at most 2,500 cars per search: add filters (make, model, year, price band) or several search URLs to go beyond. Default: `100`. |
| `includeDetails` | boolean | Also open each listing: first registration (when shown), body type, colour, doors, engine size, power, all equipment, all photos, dealer name and place, and AutoUncle's exact market value for every rated car. Adds $1.00 per 1,000 listings. About 3 s extra per 10 listings. Default: `false`. |
| `trackHistory` | boolean | Keep the price of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct and price_history. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Example: `golf-watch`. Default: `"cars-history"`. |

## Examples

**Best-priced used BMW 3-Series in Germany on AutoUncle**: Find BMW 3-Series under 30,000 euro that AutoUncle rates a good or super price (4-5 stars): price, how far it sits under AutoUncle's market value, mileage, year, fuel, power, city and days listed. Newest listings first. Export to JSON, CSV or Excel.

```json
{
  "country": "de",
  "make": "BMW",
  "model": "3-Series",
  "condition": "used",
  "max_price": 30000,
  "min_price_rating": 4,
  "limit": 200
}
```

**Electric cars in the Netherlands with price history**: Track used electric cars on autouncle.nl over time: asking price, price changes, AutoUncle price rating and market value, mileage, year, range and days listed. Run it daily to catch price drops.

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

One dataset item per result. Main fields: `title`, `year`, `mileage`, `fuel_type`, `price`, `price_rating`, `price_below_market`, `city`, `days_listed`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `autouncle:<country>:<AutoUncle car id>`, e.g. `autouncle:de:229959445` |
| `source` | string | Site short name (autouncle) |
| `url` | string | AutoUncle listing page (`/d/<id>-<slug>`), with the dealer's offer one click away |
| `title` | string | Make, model and trim as AutoUncle shows them, e.g. `VW Golf VII Comfortline` |
| `make` | string | Make, e.g. BMW |
| `model` | string | Model, e.g. X1 |
| `variant` | string | Trim after the model, e.g. `Comfortline`, `M Sport` |
| `subtitle` | string | The ad's own headline under the title, e.g. `1.0TSI 85KW DSG LED GRA PDC SHZ` (dealer wording, in the site's language) |
| `condition` | string | used or new (from the label on the photo; demo cars count as used) |
| `price` | integer | Asking price in the site's currency (see price_currency) |
| `price_currency` | string | ISO currency code of the country's site: EUR, GBP, CHF, DKK, SEK, PLN, RON |
| `price_type` | string | fixed or on_request (no price shown) |
| `price_rating` | string | AutoUncle's price rating vs its market value: very_good (Superpreis, 5 stars), good (4), fair (3), increased (2), high (1). Null when AutoUncle does not rate the car (e.g. rare vehicles) |
| `price_rating_score` | integer | AutoUncle's price rating as a number: 5 (Superpreis) ... 1 (Teuer), the stars on the site |
| `price_rating_label` | string | AutoUncle's rating text in the site's language, e.g. `Guter Preis`, `Goede deal` |
| `market_price` | integer | AutoUncle's market value for comparable cars, in price_currency. List results: price + price_below_market (only rated 3-5 stars); Full details add-on: the exact value for every rated car |
| `price_below_market` | integer | How far the price is under AutoUncle's market value (rated 3-5 stars only; null above market) |
| `listing_price_change_pct` | number | Price change since the car was listed, in percent (negative = cheaper), as AutoUncle shows it |
| `days_listed` | integer | Days the car has been advertised (0 = today) |
| `mileage` | integer | Mileage in mileage_unit (Swedish `mil` is converted to km) |
| `mileage_unit` | string | km, or mi on autouncle.co.uk |
| `year` | integer | Year of first registration |
| `first_registration` | string | Details add-on: first registration, YYYY-MM (when the listing page has it) |
| `fuel_type` | string | petrol, diesel, electric, hybrid (any mix of two fuels), lpg, cng, hydrogen, ethanol or other |
| `transmission` | string | manual or automatic (null when the card shows no gearbox, e.g. electric cars) |
| `body_type` | string | hatchback, sedan, wagon, suv, coupe, convertible, minivan, van or pickup (null when unclear) |
| `power_kw` | integer | Engine power in kW |
| `power_hp` | integer | Engine power in hp (PS / PK / CV / HP as the site's country uses it) |
| `electric_range_km` | integer | Range of an electric car (WLTP) as shown on the card |
| `engine_cc` | integer | Details add-on: engine size in cc |
| `color` | string | Details add-on: colour, in the site's language |
| `doors` | integer | Details add-on: number of doors |
| `seats` | integer | Not published by AutoUncle (null) |
| `vin` | string | Not published by AutoUncle (null) |
| `seller_type` | string | dealer or private. Dealers are recognised on the card, everything else from the Seller filter or the Full details add-on; otherwise null (AutoUncle also lists marketplace ads) |
| `seller_name` | string | Dealer name (null for marketplace and private ads, see source_site) |
| `source_site` | string | Where the ad lives: the dealer, or a marketplace such as Kleinanzeigen, AutoScout24, Pkw.de, Marktplaats |
| `seller_phone` | string | Not published by AutoUncle (null) |
| `seller_url` | string | Not published by AutoUncle (null) |
| `address` | string | Place as AutoUncle shows it: postcode, city, region |
| `city` | string | City |
| `postcode` | string | Postcode |
| `region` | string | Region / state / county |
| `country` | string | Details add-on: ISO country code of the car |
| `lat` | number | Not published by AutoUncle (null) |
| `lon` | number | Not published by AutoUncle (null) |
| `listed_at` | string | Not published by AutoUncle (null); see days_listed |
| `image_urls` | array | Photo URLs: the first photo, all photos with the Full details add-on |
| `description` | string | Not published by AutoUncle (null); see subtitle |
| `equipment` | array | Details add-on: equipment and options, in the site's language |
| `details_fetched` | boolean | True when the Full details add-on enriched this item |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: true when the status changed |
| `price_history` | array | Price history add-on: price points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $0.50 per 1,000 car listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One car listing enriched with its listing page: first registration (when shown), body type, colour, doors, engine size, power, all equipment, all photos, dealer name and place, and AutoUncle's exact market value for every rated car.
- Price history: $1.00 per 1,000. One car listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [AutoUncle Car Listings Scraper](https://apify.com/carabain-digital/autouncle-listings?fpr=4ijnom)

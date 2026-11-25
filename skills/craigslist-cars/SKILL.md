---
name: craigslist-cars
description: "Scrape cars and trucks for sale from Craigslist (US): price, year, make, model, mileage, dealer or owner, location, photos. Search by city area, make, model, price, year, mileage, fuel. Newest first. Add-ons: full details with VIN and description, price history. Runs the Apify Actor carabain-digital/craigslist-cars. Use for tasks like: Craigslist cars by owner under $10,000 in the SF Bay Area; Toyota Tacoma on Craigslist Portland with price history."
---

# Craigslist Cars & Trucks Scraper

Scrape cars and trucks for sale from Craigslist (US): price, year, make, model, mileage, dealer or owner, location, photos. Search by city area, make, model, price, year, mileage, fuel. Newest first. Add-ons: full details with VIN and description, price history.

Runs on Apify as the Actor [`carabain-digital/craigslist-cars`](https://apify.com/carabain-digital/craigslist-cars?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 6,250 car listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~craigslist-cars/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~craigslist-cars/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"location": "sfbay", "seller_type": "private", "max_price": 10000, "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/craigslist-cars").call(run_input={"location": "sfbay", "seller_type": "private", "max_price": 10000, "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/craigslist-cars`.

## Input

| Field | Type | Description |
|---|---|---|
| `location` | string | The Craigslist area to search: the subdomain of https://<area>.craigslist.org or the city name. Craigslist has no nationwide search, one run searches one area (use Search URLs for several). Example: `sfbay`, `newyork`, `losangeles`, `chicago`, `houston`, `seattle`, `Portland, OR`. Ignored when you give search URLs. |
| `make` | string | Car make, matched as search text on the make and model field and the title. Example: `Toyota`, `Ford`, `Honda`. |
| `model` | string | Model of that make (needs Make). Example: `Camry`, `F-150`, `Civic`. |
| `keyword` | string | Free-text search over the listing title and text, as the search box on the site. Example: `manual`, `low miles`, `4x4`. |
| `condition` | string | Craigslist's condition attribute is optional, so `used` and `new` only return listings whose seller set it. Example: `any`. Values: `any`, `used`, `new`. Default: `"any"`. |
| `min_year` | integer | Oldest model year. Example: `2018`. |
| `max_year` | integer | Newest model year. Example: `2023`. |
| `min_price` | integer | Lowest price in US dollars. Example: `10000`. |
| `max_price` | integer | Highest price in US dollars. Example: `25000`. |
| `max_mileage` | integer | Highest odometer reading in miles. Example: `60000`. |
| `fuel_type` | string | Fuel type as Craigslist offers it. Example: `electric`. Values: `any`, `petrol`, `diesel`, `hybrid`, `electric`, `other`. Default: `"any"`. |
| `transmission` | string | Transmission. Example: `manual`. Values: `any`, `automatic`, `manual`. Default: `"any"`. |
| `body_type` | string | Body type. Craigslist also has `truck` (large trucks), that is part of `other` here. Example: `suv`. Values: `any`, `sedan`, `suv`, `pickup`, `hatchback`, `coupe`, `convertible`, `wagon`, `minivan`, `van`, `other`. Default: `"any"`. |
| `seller_type` | string | `private` = Craigslist's by-owner listings, `dealer` = by-dealer. Example: `private`. Values: `any`, `private`, `dealer`. Default: `"any"`. |
| `start_urls` | array | Cars & trucks search pages copied from craigslist.org; they replace the filters above and can cover several areas. Example: `https://sfbay.craigslist.org/search/cta?purveyor=owner&min_price=2000&max_price=9000`. Filters in the URL (price, year, mileage, fuel, body, transmission, make and model, by owner or by dealer) are kept; results always come newest first. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `100`. Craigslist serves at most 10,000 listings per search. Default: `100`. |
| `includeDetails` | boolean | Also open each listing: full description, all photos, fuel, transmission, body type, colour, drivetrain, cylinders, title status, condition, VIN, exact post and update time, town, ZIP code and coordinates. Adds $1.00 per 1,000 listings on top of the base price. One extra request per listing (about 1 second per 7 listings). Default: `false`. |
| `trackHistory` | boolean | Keep the price of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `camry-nyc`. Default: `"cars-history"`. |

## Examples

**Craigslist cars by owner under $10,000 in the SF Bay Area**: Scrape the newest by-owner cars and trucks under $10,000 from Craigslist in the SF Bay Area: price, year, make, model, mileage, location and photos. Run it on a schedule to catch fresh deals before others do. Export to JSON, CSV or Excel.

```json
{
  "location": "sfbay",
  "seller_type": "private",
  "max_price": 10000,
  "limit": 100
}
```

**Toyota Tacoma on Craigslist Portland with price history**: Scrape Toyota Tacoma listings from Craigslist Portland, owners and dealers, with VIN, mileage and full description, and track their price over time. Run it daily to spot price drops and new listings.

```json
{
  "location": "portland",
  "make": "Toyota",
  "model": "Tacoma",
  "min_year": 2005,
  "limit": 100,
  "includeDetails": true,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `title`, `price`, `mileage`, `year`, `city`, `region`, `seller_type`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `craigslist:<post id>` |
| `source` | string | Short site name (cargurus) |
| `url` | string | Public listing URL (craigslist.org/view/d/...) |
| `title` | string | Listing title as the seller wrote it |
| `make` | string | Make read from the title (known make list; with details: the seller's make field when the title has none) |
| `model` | string | Model: the word after the make in the title (two words for `Model 3`, `3 Series`) |
| `variant` | string | The rest of the title after the model (trim, extras, seller text) |
| `condition` | string | Details add-on: `new` or `used` (Craigslist's condition attribute; null when the seller left it out) |
| `price` | number | Asking price in USD; null when under $100 (Craigslist placeholders such as $1) |
| `price_currency` | string | USD |
| `price_type` | string | fixed, or on_request when there is no price |
| `mileage` | integer | Odometer in miles (as the seller entered it) |
| `mileage_unit` | string | mi (miles) |
| `year` | integer | Model year |
| `first_registration` | string | Always null: US listings only have the model year |
| `fuel_type` | string | Details add-on: petrol, diesel, hybrid, electric or other |
| `transmission` | string | Details add-on: manual or automatic |
| `body_type` | string | Details add-on: sedan, suv, pickup (incl. Craigslist `truck`), hatchback, coupe, convertible, wagon, minivan, van or other |
| `power_kw` | integer | Engine power in kW (from hp) |
| `power_hp` | integer | Engine power in hp, when CarGurus knows it |
| `engine_cc` | integer | Engine displacement in cc |
| `color` | string | Details add-on: paint colour |
| `doors` | integer | Number of doors |
| `seats` | integer | Always null: not published |
| `vin` | string | Details add-on: 17-character VIN when the seller entered it |
| `seller_type` | string | dealer (Craigslist `by dealer`) or private (`by owner`) |
| `seller_name` | string | Always null: Craigslist does not publish seller names |
| `seller_phone` | string | Always null: Craigslist relays replies and hides phone numbers (dealers often write theirs in the description) |
| `seller_url` | string | Details add-on: dealer website |
| `address` | string | Details add-on: `<town>, <state>` from the listing's location |
| `city` | string | The place the seller typed (town or neighbourhood, e.g. `willow glen / cambrian`); the town from the listing with details |
| `postcode` | string | Details add-on: ZIP code when the seller set one |
| `region` | string | US state (of the Craigslist area; the listing's state with details) |
| `country` | string | US |
| `lat` | number | Latitude of the listing location (approximate) |
| `lon` | number | Longitude of the listing location (approximate) |
| `listed_at` | string | When the listing was posted or last reposted/bumped, UTC ISO 8601 (the date Craigslist sorts by) |
| `image_urls` | array | Photo URLs (600x450), all of them |
| `description` | string | Details add-on: the listing text |
| `area` | string | Craigslist area (subdomain) searched, e.g. `sfbay` |
| `subarea` | string | Sub-area abbreviation inside the area, e.g. `sby` (south bay) |
| `post_id` | integer | Craigslist post id |
| `uuid` | string | Craigslist listing key used in the URL |
| `drivetrain` | string | Details add-on: fwd, rwd or 4wd |
| `cylinders` | string | Details add-on: e.g. `6 cylinders` |
| `title_status` | string | Details add-on: clean, salvage, rebuilt, parts only, lien or missing |
| `body_type_original` | string | Details add-on: Craigslist's own body type (bus, offroad, truck, ...) |
| `make_model_text` | string | Details add-on: the make and model as the seller typed them |
| `updated_at` | string | Details add-on: last update time, UTC ISO 8601 |
| `details_fetched` | boolean | True when the details add-on enriched this item |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: always false for CarGurus (no status field) |
| `price_history` | array | Price history add-on: price points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $0.80 per 1,000 car listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One listing enriched with its listing page: full description, all photos, fuel, transmission, body type, colour, drivetrain, cylinders, title status, condition, VIN, exact post time, town, ZIP code and coordinates.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [Craigslist Cars & Trucks Scraper](https://apify.com/carabain-digital/craigslist-cars?fpr=4ijnom)

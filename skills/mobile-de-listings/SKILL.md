---
name: mobile-de-listings
description: "Scrape car listings from mobile.de, Germany's largest car marketplace: make, model, price, mileage, year, fuel, power, seller, photos and mobile.de's price rating. Filter by make, model, price, year, km, fuel, body, location. Newest first. Add-ons: full details, price history. Runs the Apify Actor carabain-digital/mobile-de-listings. Use for tasks like: Used BMW 3 Series under 25,000 EUR on mobile.de; Tesla Model 3 prices over time on mobile.de."
---

# mobile.de Car Listings Scraper

Scrape car listings from mobile.de, Germany's largest car marketplace: make, model, price, mileage, year, fuel, power, seller, photos and mobile.de's price rating. Filter by make, model, price, year, km, fuel, body, location. Newest first. Add-ons: full details, price history.

Runs on Apify as the Actor [`carabain-digital/mobile-de-listings`](https://apify.com/carabain-digital/mobile-de-listings). The user needs an Apify account and an API token ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)); runs are billed to their account.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~mobile-de-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~mobile-de-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"make": "BMW", "model": "3er", "condition": "used", "min_year": 2018, "max_price": 25000, "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/mobile-de-listings").call(run_input={"make": "BMW", "model": "3er", "condition": "used", "min_year": 2018, "max_price": 25000, "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/mobile-de-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `make` | string | Car make as mobile.de lists it. Example: `BMW`, `Volkswagen` (or `VW`), `Mercedes-Benz`, `Audi`, `Tesla`. Leave empty for all makes. Ignored when you give search URLs. |
| `model` | string | Model or model series of that make. Example: `Golf`, `3er` (BMW 3 Series: all 316 ... 340), `320`, `C-Klasse`, `Model 3`. A name mobile.de does not know is searched as the seller's model text. Needs a make. |
| `min_year` | integer | First registration from this year on. Example: `2018`. |
| `max_year` | integer | First registration up to this year. Example: `2022`. |
| `min_price` | integer | Lowest price in euro (gross). Example: `5000`. |
| `max_price` | integer | Highest price in euro (gross). Example: `25000`. |
| `max_mileage` | integer | Highest mileage in kilometres. Example: `100000`. |
| `fuel_type` | string | Fuel. `hybrid` includes plug-in hybrids; `plugin_hybrid` keeps only plug-ins. Example: `electric`. Values: `any`, `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`, `lpg`, `cng`, `hydrogen`, `ethanol`, `other`. Default: `"any"`. |
| `transmission` | string | Gearbox. Example: `automatic`. Values: `any`, `manual`, `automatic`. Default: `"any"`. |
| `body_type` | string | mobile.de category. Pickups are in the SUV category and minivans in Van/Minibus. Example: `suv`. Values: `any`, `sedan`, `hatchback`, `wagon`, `suv`, `pickup`, `coupe`, `convertible`, `van`, `minivan`, `other`. Default: `"any"`. |
| `condition` | string | New or used cars. Example: `used`. Values: `any`, `used`, `new`. Default: `"any"`. |
| `seller_type` | string | Dealers or private sellers. Example: `dealer`. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `location` | string | German postcode or city to search around. Example: `10115`, `München`, `Köln`. Leave empty for all of Germany. |
| `radius_km` | integer | Search radius around the location in km (default 50). Example: `25`. |
| `start_urls` | array | Search result pages copied from suchen.mobile.de; they replace the filters above. Example: `https://suchen.mobile.de/fahrzeuge/search.html?isSearchRequest=true&ms=3500%3B%3B21%3B&p=%3A25000&s=Car&vc=Car`. Filters in the URL are kept; the page number is ignored; results come newest first unless the URL sorts otherwise. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `100`. mobile.de serves 2,000 results per search; above that the Actor splits the search into price bands (cheapest band first, newest first within each band). Default: `100`. |
| `includeDetails` | boolean | Also open each listing: seller description, equipment list, every attribute (previous owners, HU, climate control, emission sticker, consumption, CO2 ...), all photos, all seller phones and WhatsApp, contact person, opening hours, dealer ratings, KBA codes and mobile.de's AI summary. Adds $0.50 per 1,000 listings. One extra request per listing (about 10 s per 100 listings). Default: `false`. |
| `trackHistory` | boolean | Keep the price of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `golf-berlin`. Default: `"cars-history"`. |

## Examples

**Used BMW 3 Series under 25,000 EUR on mobile.de**: Scrape the newest used BMW 3 Series (316 to 340) listed on mobile.de under 25,000 euro from 2018 on: price, mileage, year, power, fuel, seller and mobile.de's price rating. Export to JSON, CSV or Excel.

```json
{
  "make": "BMW",
  "model": "3er",
  "condition": "used",
  "min_year": 2018,
  "max_price": 25000,
  "limit": 100
}
```

**Tesla Model 3 prices over time on mobile.de**: Track the prices of used Tesla Model 3 listings on mobile.de over time: price, price drops, mileage, year, seller and price rating. Run it daily to catch price drops.

```json
{
  "make": "Tesla",
  "model": "Model 3",
  "limit": 500,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `make`, `model`, `year`, `mileage`, `fuel_type`, `price`, `price_rating`, `city`, `seller_type`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `mobilede:<ad id>` |
| `source` | string | Short site name (mobilede) |
| `url` | string | Public listing URL on suchen.mobile.de |
| `title` | string | Listing title (make, model and the seller's model text) |
| `make` | string | Make, e.g. BMW |
| `model` | string | Model as mobile.de classifies it, e.g. 320 |
| `variant` | string | The seller's model description, e.g. 320d xDrive M Sport |
| `condition` | string | used, new or demo |
| `price` | number | Price in EUR (gross, incl. VAT) |
| `price_currency` | string | ISO currency code (EUR) |
| `price_type` | string | fixed, negotiable (VB) or on_request |
| `mileage` | number | Mileage |
| `mileage_unit` | string | km |
| `year` | integer | Year of first registration (new cars: model year) |
| `first_registration` | string | First registration, YYYY-MM |
| `fuel_type` | string | petrol, diesel, electric, hybrid, plugin_hybrid, lpg, cng, hydrogen, ethanol or other |
| `transmission` | string | manual or automatic |
| `body_type` | string | sedan, hatchback (Kleinwagen), wagon, suv (incl. pickups), coupe, convertible, van or other |
| `power_kw` | integer | Power in kW |
| `power_hp` | integer | Power in PS (hp) |
| `engine_cc` | number | Engine displacement in cm3 |
| `color` | string | Colour (details add-on: with finish, e.g. Blau Metallic) |
| `doors` | integer | Number of doors (4/5 -> 5) |
| `seats` | integer | Number of seats |
| `vin` | string | Always null: mobile.de does not publish VINs |
| `seller_type` | string | dealer or private |
| `seller_name` | string | Dealer name (private sellers stay anonymous) |
| `seller_phone` | string | Seller phone number |
| `seller_url` | string | Dealer page on mobile.de |
| `address` | string | Seller street and postcode + city |
| `city` | string | City of the vehicle |
| `postcode` | string | German postcode |
| `region` | string | Always null |
| `country` | string | ISO country code of the vehicle location |
| `lat` | number | Latitude of the seller |
| `lon` | number | Longitude of the seller |
| `listed_at` | string | When the ad went online or was last renewed (mobile.de's 'online since'), UTC ISO 8601 |
| `image_urls` | array | Photo URLs (1024 px) |
| `description` | string | Details add-on: the seller's description text (German) |
| `status` | string | available |
| `category` | string | mobile.de category label, e.g. SUV/Geländewagen/Pickup |
| `price_net` | number | Net price for dealers that show VAT |
| `vat_rate` | number | VAT rate in percent (dealers) |
| `price_rating` | string | mobile.de price rating vs market: very_good_price, good_price, reasonable_price, increased_price, high_price or no_rating |
| `price_rating_label` | string | Price rating as mobile.de shows it, e.g. Guter Preis |
| `price_rating_thresholds` | array | mobile.de's price band edges in EUR for this car (very good ... high) |
| `previous_owners` | integer | Number of previous owners |
| `next_inspection` | string | Next HU (TÜV) inspection, YYYY-MM or text like Neu |
| `emission_class` | string | Emission class, e.g. Euro6d |
| `has_damage` | boolean | True when the car is damaged |
| `ready_to_drive` | boolean | Fahrtauglich flag |
| `num_images` | integer | Number of photos |
| `seller_rating` | number | Dealer rating (1-5) |
| `seller_rating_count` | integer | Number of dealer ratings |
| `seller_id` | integer | mobile.de seller/customer id |
| `leasing` | object | Leasing offer of new cars: lender and rates |
| `highlights` | array | Seller highlights shown on the ad |
| `created_at` | string | When the ad was first created, UTC ISO 8601 |
| `modified_at` | string | Last change of the ad, UTC ISO 8601 |
| `details_fetched` | boolean | True when the Full details add-on enriched this item |
| `features` | array | Details add-on: equipment list (German), e.g. Navigationssystem, Sitzheizung |
| `attributes` | object | Details add-on: every labelled attribute (German label -> value): HU, Klimatisierung, Umweltplakette, consumption, CO2 ... |
| `ai_summary` | string | Details add-on: mobile.de's own summary of the ad (German) |
| `damage_condition` | string | Details add-on: e.g. Gebrauchtfahrzeug, Unfallfrei |
| `seller_phones` | array | Details add-on: all seller phone numbers |
| `seller_whatsapp` | string | Details add-on: WhatsApp number |
| `seller_contact_person` | string | Details add-on: dealer contact person |
| `seller_opening_hours` | array | Details add-on: dealer opening hours |
| `seller_since` | string | Details add-on: 'Bei mobile.de seit ...' |
| `seller_recommendation_rate` | number | Details add-on: dealer recommendation rate in percent |
| `kba_hsn` | string | KBA manufacturer key (HSN), when the seller gives it |
| `kba_tsn` | string | KBA type key (TSN), when the seller gives it |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: true when the status changed since the previous run |
| `price_history` | array | Price history add-on: `[{at, price, status}]`, last 20 points |

## Price

Pay per result: $0.50 per 1,000 car listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $0.50 per 1,000. One listing enriched from its ad page: description, equipment list, all attributes (previous owners, HU, consumption, CO2 ...), all photos, all seller phones and WhatsApp, contact person, opening hours, dealer ratings, KBA codes and mobile.de's AI summary.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [mobile.de Car Listings Scraper](https://apify.com/carabain-digital/mobile-de-listings)

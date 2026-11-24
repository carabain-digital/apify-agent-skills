---
name: belgium-cars
description: "Search AutoScout24, Gocar and 2dehands at once for used and new cars in Belgium. One search, one dataset: merged, deduplicated on VIN or make, year, mileage and price, newest first. Optional full details and price history. Residential proxy included. Export to JSON, CSV or Excel. Runs the Apify Actor carabain-digital/belgium-cars. Use for tasks like: Used Volkswagen Golf in Belgium from three car sites; Electric cars in Belgium with price history."
---

# Belgium Car Listings Scraper

Search AutoScout24, Gocar and 2dehands at once for used and new cars in Belgium. One search, one dataset: merged, deduplicated on VIN or make, year, mileage and price, newest first. Optional full details and price history. Residential proxy included. Export to JSON, CSV or Excel.

Runs on Apify as the Actor [`carabain-digital/belgium-cars`](https://apify.com/carabain-digital/belgium-cars?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 6,250 car listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~belgium-cars/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~belgium-cars/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"make": "Volkswagen", "model": "Golf", "condition": "used", "limit": 200}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/belgium-cars").call(run_input={"make": "Volkswagen", "model": "Golf", "condition": "used", "limit": 200})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/belgium-cars`.

## Input

| Field | Type | Description |
|---|---|---|
| `sites` | array | Which sites to search. Default: all three. Example: `["autoscout24", "gocar"]`. Default: `["autoscout24", "gocar", "2dehands"]`. |
| `make` | string | Car make. Example: `Volkswagen`, `BMW`, `Mercedes-Benz`, `Land Rover`. Leave empty for all makes. |
| `model` | string | Model of that make (needs a make). Example: `Golf`, `X1`, `A4`. Sites spell some models differently (BMW `3 Series` on AutoScout24, `3 Reeks` on Gocar and 2dehands): a site that does not know the model is skipped with a message in the log and the others still run. |
| `min_year` | integer | Earliest year of first registration. Example: `2018`. New cars without a registration are left out when set. |
| `max_year` | integer | Latest year of first registration. Example: `2022`. |
| `min_price` | integer | Lowest price in euro. Example: `5000`. |
| `max_price` | integer | Highest price in euro. Example: `25000`. |
| `max_mileage` | integer | Highest mileage in km. Example: `100000`. |
| `fuel_type` | string | Fuel type. Example: `electric`. AutoScout24 and Gocar do not separate plug-in hybrids, so `plugin_hybrid` returns all hybrids there. Values: `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`, `lpg`, `cng`, `hydrogen`, `ethanol`, `other`. |
| `transmission` | string | Gearbox. Example: `automatic`. Values: `automatic`, `manual`. |
| `body_type` | string | Body type. Example: `suv`. 2dehands has no `van` or `pickup` body: for those it is skipped. Values: `hatchback`, `sedan`, `wagon`, `suv`, `coupe`, `convertible`, `minivan`, `van`, `pickup`, `other`. |
| `condition` | string | New or used cars. Example: `used`. Values: `any`, `used`, `new`. Default: `"any"`. |
| `seller_type` | string | Dealer or private seller. Example: `dealer`. Gocar is ~99% dealers. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `location` | string | Search around a Belgian postcode. Example: `9000` (Gent), `2000` (Antwerpen), `1000` (Brussel). Use a 4-digit postcode: 2dehands only takes postcodes (a city name skips it), AutoScout24 and Gocar also take city names. Several places separated by commas (`9000, 2000`) run each site once per place. |
| `radius_km` | integer | Distance around the location, in km. Example: `25`. Only used with a location. |
| `limit` | integer | Cap on the merged dataset, newest first. Example: `100`. Each site is searched up to this many, then the sites are mixed evenly and deduplicated. Default: `100`. |
| `limit_per_site` | integer | Optional cap per site, to keep the mix balanced. Default: Max listings. Example: `50`. |
| `includeDetails` | boolean | Also open each listing's detail page on its site: full description, all photos, equipment, colour, doors, seats, engine, previous owners, Car-Pass link, dealer phone and website, exact listing date and more. Adds $1.00 per 1,000 listings that were actually enriched. Runs take longer. Default: `false`. |
| `trackHistory` | boolean | Keep a price history per car in a named storage in your own account and add `first_seen_at`, `previous_price`, `price_change`, `price_change_pct` and `price_history` to every listing you have scraped before. The history store is shared with our single-site car Actors. Adds $1.00 per 1,000 listings that already had history (the first run only starts the series). Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (a-z, 0-9 and dashes). Use the same name in all our car Actors to share one history. Example: `cars-history`. Default: `"cars-history"`. |

## Examples

**Used Volkswagen Golf in Belgium from three car sites**: The newest used Volkswagen Golf listings from AutoScout24, Gocar and 2dehands in one deduplicated list, with price, mileage, year, fuel, seller and city.

```json
{
  "make": "Volkswagen",
  "model": "Golf",
  "condition": "used",
  "limit": 200
}
```

**Electric cars in Belgium with price history**: The newest electric cars for sale in Belgium from AutoScout24, Gocar and 2dehands, deduplicated, with price changes since your last run. Schedule it daily for a price-drop feed.

```json
{
  "fuel_type": "electric",
  "limit": 500,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `source`, `title`, `year`, `mileage`, `fuel_type`, `price`, `city`, `seller_type`, `also_on`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `autoscout24:<listing uuid>` |
| `source` | string | Site the item was taken from: autoscout24, gocar, 2dehands |
| `url` | string | Listing page on AutoScout24 |
| `title` | string | Make, model and trim, e.g. Volkswagen Golf 1.5 TSI Life |
| `make` | string | Make, e.g. BMW |
| `model` | string | Model, e.g. X1 |
| `variant` | string | Trim / version as the seller wrote it, e.g. xDrive28i M Sport |
| `condition` | string | used, new, demo (demonstrator or pre-registered) or classic |
| `price` | integer or number | Asking price in EUR (gross, incl. VAT) |
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
| `country` | string | ISO country code of the car's location (BE; AutoScout24 also lists cars from neighbouring countries near the border) |
| `lat` | number | Details add-on: latitude |
| `lon` | number | Details add-on: longitude |
| `listed_at` | string | When the listing was published, ISO 8601 (the list is sorted on it, newest first). AutoScout24 search results have none (only with the Full details add-on): those come after the dated listings, in AutoScout24's own newest-first order |
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
| `co2_g_km` | integer or number | Details add-on: CO2 emissions, g/km |
| `net_price` | integer | Details add-on: price excl. VAT, for VAT-deductible cars |
| `vat_deductible` | boolean | Details add-on: VAT deductible (dealer cars) |
| `market_price_median` | integer | Details add-on: AutoScout24's market price for comparable cars (median) |
| `price_rating_bands` | array | Details add-on: price bands behind the rating, `[[min, max], ...]` from very good to high |
| `views` | integer | Details add-on: listing page views |
| `favorites` | integer | Details add-on: times saved as favourite |
| `seller_website` | string | Details add-on: dealer's own website |
| `fuel_type_original` | string | Fuel as the site shows it, e.g. Hybride - Benzine |
| `body_type_original` | string | Body style as the site shows it, e.g. Break/Stationwagon |
| `drive` | string | front, rear or awd when the seller filled it in |
| `emission_class` | string | Euro emission class, e.g. Euro 6d |
| `battery_capacity_kwh` | number | Battery capacity of electric cars in kWh |
| `warranty_months` | integer | Warranty in months |
| `is_leasing` | boolean | True for leasing offers |
| `lease_price_monthly` | number | Monthly lease price in EUR for leasing offers |
| `has_carpass` | boolean | True when a Car-Pass (Belgian mileage history) is available |
| `carpass_url` | string | Details add-on: link to the public Car-Pass mileage history report |
| `damaged` | boolean | True when the car is listed as damaged |
| `vehicle_type` | string | car, van or classic |
| `images_count` | integer | Number of photos |
| `seller_id` | integer | Gocar dealer (point of sale) ID |
| `registration_date` | string | Details add-on: exact first registration date, YYYY-MM-DD |
| `interior_color` | string | Details add-on: interior colour |
| `upholstery` | string | Details add-on: upholstery, e.g. Leder |
| `inspection_valid_until` | string | Details add-on: technical inspection valid until, as the site writes it |
| `specs` | object | Details add-on: the full spec table as site label -> value (null = not filled in) |
| `promoted` | boolean | Paid 'Dagtopper' ad (bumped to the top daily) |
| `reserved` | boolean | Marked as reserved |
| `energy_label` | string | Energy label (NL) |
| `car_pass_url` | string | Public Car-Pass mileage certificate link (2dehands) |
| `category` | string | The site's car section (make, or Bestelauto's/Vrachtwagens/Oldtimers) |
| `license_plate` | string | Details add-on: licence plate (NL) |
| `apk_until` | string | Details add-on: APK (MOT) expiry date (NL) |
| `imported` | boolean | Details add-on: imported car (NL) |
| `seller_active_years` | integer | Details add-on: years the seller has been on the site |
| `service_history` | string | Details add-on: maintenance history, e.g. Dealer onderhouden |
| `new_price_text` | string | Details add-on: list price when new, as the site shows it (NL) |
| `empty_weight_kg` | integer | Details add-on: empty weight in kg |
| `nap_status` | boolean | Details add-on: NAP odometer check available (NL) |
| `view_count` | integer | Details add-on: page views |
| `favorite_count` | integer | Details add-on: times saved as favourite |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: true when the status changed |
| `price_history` | array | Price history add-on: price points over your runs, `[{at, price, status}]`, last 20 |
| `also_on` | array | Other sites that carry the same car (same VIN, or same make, year, mileage and price); empty when only one does |

## Price

Pay per result: $0.80 per 1,000 car listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. Opens each listing's detail page on AutoScout24, Gocar or 2dehands: full description, all photos, equipment, colour, engine, previous owners, Car-Pass link, dealer phone and website, exact listing date and more.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [Belgium Car Listings Scraper](https://apify.com/carabain-digital/belgium-cars?fpr=4ijnom)

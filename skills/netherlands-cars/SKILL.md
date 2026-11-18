---
name: netherlands-cars
description: "Search AutoScout24, Marktplaats, AutoTrack, AutoWereld, Gaspedaal and AutoUncle for used and new cars in the Netherlands. One search, one dataset: merged, deduplicated across sites, newest first. Optional full details and price history. Export to JSON, CSV or Excel. Runs the Apify Actor carabain-digital/netherlands-cars. Use for tasks like: Volkswagen Golf for sale in the Netherlands; Dutch used cars under 15,000 with price history."
---

# Netherlands Car Listings Scraper

Search AutoScout24, Marktplaats, AutoTrack, AutoWereld, Gaspedaal and AutoUncle for used and new cars in the Netherlands. One search, one dataset: merged, deduplicated across sites, newest first. Optional full details and price history. Export to JSON, CSV or Excel.

Runs on Apify as the Actor [`carabain-digital/netherlands-cars`](https://apify.com/carabain-digital/netherlands-cars), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 5,555 car listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~netherlands-cars/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~netherlands-cars/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"make": "Volkswagen", "model": "Golf", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/netherlands-cars").call(run_input={"make": "Volkswagen", "model": "Golf", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/netherlands-cars`.

## Input

| Field | Type | Description |
|---|---|---|
| `make` | string | Car make, sent to every selected site. Example: `Volkswagen`, `BMW`, `Mercedes-Benz`, `Land Rover`. Leave empty for all makes. |
| `model` | string | Model within the make (needs `make`), as the Dutch sites spell it. Example: `Golf`, `Polo`, `C-Klasse`, `3-Serie` (BMW series are translated to each site's own spelling). A site that does not know the model is skipped and logged. |
| `min_year` | integer | Oldest construction year. Example: `2015`. |
| `max_year` | integer | Newest construction year. Example: `2022`. |
| `min_price` | integer | Lowest asking price in euro. Example: `5000`. |
| `max_price` | integer | Highest asking price in euro. Example: `20000`. |
| `max_mileage` | integer | Highest odometer reading in km. Example: `150000`. |
| `fuel_type` | string | Fuel type. Example: `diesel`. AutoScout24 and Gaspedaal have one hybrid category, so `plugin_hybrid` returns all hybrids there. Values: `any`, `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`, `lpg`, `cng`, `hydrogen`, `other`. Default: `"any"`. |
| `transmission` | string | Gearbox. Example: `automatic`. Values: `any`, `manual`, `automatic`. Default: `"any"`. |
| `body_type` | string | Body type. Example: `suv`. Marktplaats has no `van`/`pickup` filter (vans are a separate section there): a site that does not know the body type is skipped. Values: `any`, `hatchback`, `sedan`, `wagon`, `suv`, `coupe`, `convertible`, `minivan`, `van`, `pickup`, `other`. Default: `"any"`. |
| `condition` | string | New or used cars. Example: `used`. Gaspedaal lists used cars only and is skipped for `new`. Values: `any`, `used`, `new`. Default: `"any"`. |
| `seller_type` | string | Dealers or private sellers. Example: `private`. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `location` | string | Dutch postcode to search around (4 digits, optionally with the 2 letters), sent to every selected site. Example: `3511` or `1012AB`. AutoScout24 and AutoTrack also take a town (`Amersfoort`); Marktplaats and Gaspedaal only search by postcode and are skipped for a town name. Several postcodes separated by commas run as separate searches. Leave empty for all of the Netherlands. |
| `radius_km` | integer | Distance around the postcode, in km. Example: `25`. Needs a postcode; when empty each site uses its own default (10 to 50 km). |
| `sites` | array | Which sites to search. Default: AutoScout24, Marktplaats and AutoTrack. AutoWereld (dealers) opens a browser for its consent gate, which passes on only some proxy exits: a run can take a minute or more and the site is skipped when no exit passes. Gaspedaal and AutoUncle collect cars from the other sites and dealer sites, so most of their cars are duplicates; AutoUncle adds a price rating and market value. Example: `["autoscout24", "marktplaats", "autotrack", "autowereld"]`. Default: `["autoscout24", "marktplaats", "autotrack"]`. |
| `limit` | integer | Cap on the merged dataset, newest first; every site gets a fair share. Example: `100`. Default: `100`. |
| `limit_per_site` | integer | Optional cap per site, to keep the mix balanced. Default: Max cars. Example: `50`. |
| `includeDetails` | boolean | Also open each car's listing page on its site: description, all photos, equipment, colour, doors, seats, engine, previous owners, licence plate, APK date, coordinates and more (what each site has). Adds $1.00 per 1,000 cars that were actually enriched. One extra request per car (a few seconds per 100 cars). Default: `false`. |
| `trackHistory` | boolean | Keep the price of every car in a named store in your own Apify account and add `first_seen_at`, `previous_price`, `price_change`, `price_change_pct`, `status_changed` and `price_history` to each car you scraped before. The store is shared with our single-site car Actors. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 cars that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Example: `golf-watch`. Default: `"cars-history"`. |

## Examples

**Volkswagen Golf for sale in the Netherlands**: The newest Volkswagen Golf listings from AutoScout24, Marktplaats and AutoTrack in one deduplicated list, with price, mileage, year, fuel and seller.

```json
{
  "make": "Volkswagen",
  "model": "Golf",
  "limit": 100
}
```

**Dutch used cars under 15,000 with price history**: The newest used cars under 15,000 euro from AutoScout24, Marktplaats and AutoTrack, deduplicated, with price changes since your last run. Schedule it daily for a price-drop feed.

```json
{
  "condition": "used",
  "max_price": 15000,
  "limit": 500,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `source`, `title`, `year`, `mileage`, `fuel_type`, `price`, `city`, `seller_type`, `also_on`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `<source>:<site id>` |
| `source` | string | Site the car was taken from: autoscout24, marktplaats, autotrack, gaspedaal |
| `url` | string | Listing page on AutoScout24 |
| `title` | string | Make, model and trim, e.g. Volkswagen Golf 1.5 TSI Life |
| `make` | string | Make, e.g. BMW |
| `model` | string | Model, e.g. X1 |
| `variant` | string | Trim / version as the seller wrote it, e.g. xDrive28i M Sport |
| `condition` | string | used, new, demo (demonstrator or pre-registered) or classic |
| `price` | integer or number | Asking price in EUR (gross, incl. VAT) |
| `price_currency` | string | EUR |
| `price_type` | string | fixed, negotiable (details add-on) or on_request |
| `mileage` | integer or number | Mileage |
| `mileage_unit` | string | km |
| `year` | integer | Year of first registration (null for new cars) |
| `first_registration` | string | First registration, YYYY-MM |
| `fuel_type` | string | petrol, diesel, electric, hybrid, plugin_hybrid (details add-on), lpg, cng, hydrogen, ethanol or other |
| `transmission` | string | manual or automatic (semi-automatic counts as automatic) |
| `body_type` | string | Details add-on: sedan, hatchback, wagon, suv, coupe, convertible, van, minivan or other |
| `power_kw` | integer or number | Engine power in kW |
| `power_hp` | integer or number | Engine power in hp (PS) |
| `engine_cc` | integer or number | Details add-on: engine displacement in cm³ |
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
| `co2_g_km` | integer or number | Details add-on: CO2 emissions, g/km |
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
| `seller_id` | integer or string | Seller account id |
| `promoted` | boolean | Paid 'Dagtopper' ad (bumped to the top daily) |
| `reserved` | boolean | Marked as reserved |
| `energy_label` | string | Energy label (NL) |
| `emission_class` | string | Euro norm (BE) |
| `car_pass_url` | string | Public Car-Pass mileage certificate link (2dehands) |
| `fuel_type_original` | string | Fuel as the site writes it |
| `body_type_original` | string | Body type as the site writes it |
| `category` | string | The site's car section (make, or Bestelauto's/Vrachtwagens/Oldtimers) |
| `license_plate` | string | Details add-on: licence plate (NL) |
| `apk_until` | string | Details add-on: APK (MOT) expiry date (NL) |
| `imported` | boolean | Details add-on: imported car (NL) |
| `drive` | string | Details add-on: driven wheels |
| `seller_website` | string | Details add-on: dealer website |
| `seller_active_years` | integer | Details add-on: years the seller has been on the site |
| `interior_color` | string | Details add-on: interior colour |
| `upholstery` | string | Details add-on: upholstery, e.g. Leder |
| `service_history` | string | Details add-on: maintenance history, e.g. Dealer onderhouden |
| `new_price_text` | string | Details add-on: list price when new, as the site shows it (NL) |
| `empty_weight_kg` | integer | Details add-on: empty weight in kg |
| `nap_status` | boolean | Details add-on: NAP odometer check available (NL) |
| `view_count` | integer | Details add-on: page views |
| `favorite_count` | integer | Details add-on: times saved as favourite |
| `status` | string | available, reserved, sold or expected |
| `commercial_variant` | string | The dealer's own variant line |
| `vehicle_type` | string | personenauto, bedrijfswagen or overig |
| `fuel_secondary` | string | Second fuel (electric for hybrids, lpg ...) |
| `price_excl_vat` | number | Price without VAT (commercial vehicles) |
| `catalog_price` | number | New-car list price (catalogusprijs) in EUR |
| `battery_kwh` | number | Battery capacity in kWh (plug-in hybrids, EVs) |
| `mileage_verified` | boolean | NAP: RDW judged the odometer history logical |
| `fully_maintained` | boolean | Dealer states 100% maintenance history |
| `road_tax_min` | number | Road tax per quarter, lowest province (EUR) |
| `road_tax_max` | number | Road tax per quarter, highest province (EUR) |
| `seller_kind` | string | bovag_autobedrijf, officiele_merkdealer, universeel_autobedrijf ... |
| `rdw_report_url` | string | RDW vehicle report download link |
| `autoweek_report_url` | string | Autoweek report PDF |
| `video_urls` | array | Dealer video URLs |
| `cross_reference_id` | string | Listing uuid shared with AutoScout24 |
| `torque_nm` | integer | Details add-on: torque in Nm |
| `top_speed_kmh` | integer | Details add-on: top speed |
| `acceleration_0_100_s` | number | Details add-on: 0-100 km/h in seconds |
| `consumption_km_per_l` | number | Details add-on: combined consumption in km per litre (Dutch '1 op N') |
| `towing_capacity_kg` | integer | Details add-on: braked towing capacity |
| `apk_expires` | string | Details add-on: APK (MOT) valid until, YYYY-MM-DD |
| `first_owner` | boolean | Details add-on: first owner |
| `maintenance_booklet` | boolean | Details add-on: maintenance booklet present |
| `warranties` | array | Details add-on: warranties, e.g. BOVAG Garantie |
| `updated_at` | string | Details add-on: when the dealer last changed the listing |
| `source_site` | string | Gaspedaal only: The site Gaspedaal found the car on (first non-lease source with the Details add-on), e.g. AutoTrack, AutoScout24, Dealersite, ANWB |
| `source_sites` | array | Gaspedaal only: Every site that carries this car |
| `sources` | array | Gaspedaal only: Every source: `{site, type (dealer\|other\|financial), url (Gaspedaal click-out), source_url (Details add-on: the real listing URL)}` |
| `source_url` | string | Gaspedaal only: Details add-on: direct URL of the listing on the source site |
| `details_source` | string | Gaspedaal only: Details add-on: the source site the details were read from |
| `damage_history` | string | Details add-on: damage history as the dealer states it, e.g. Ja (gerepareerd) |
| `nap` | boolean | Details add-on: true when the odometer is NAP-checked |
| `subtitle` | string | The ad's own headline under the title, e.g. `1.0TSI 85KW DSG LED GRA PDC SHZ` (dealer wording, in the site's language) |
| `price_rating_score` | integer | AutoUncle's price rating as a number: 5 (Superpreis) ... 1 (Teuer), the stars on the site |
| `price_rating_label` | string | AutoUncle's rating text in the site's language, e.g. `Guter Preis`, `Goede deal` |
| `market_price` | integer | AutoUncle's market value for comparable cars, in price_currency. List results: price + price_below_market (only rated 3-5 stars); Full details add-on: the exact value for every rated car |
| `price_below_market` | integer | How far the price is under AutoUncle's market value (rated 3-5 stars only; null above market) |
| `listing_price_change_pct` | number | Price change since the car was listed, in percent (negative = cheaper), as AutoUncle shows it |
| `days_listed` | integer | Days the car has been advertised (0 = today) |
| `electric_range_km` | integer | Range of an electric car (WLTP) as shown on the card |
| `also_on` | array | Other sites that carry the same car (same VIN, else same make, year, mileage and price); empty when only one does |

## Price

Pay per result: $0.90 per 1,000 car listings on the Apify Free plan, less on paid plans, plus $0.025 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. Opens each car's listing page on its site: description, all photos, equipment, colour, doors, seats, engine, previous owners, licence plate, APK date, coordinates, dealer phone and more. Only enriched cars are charged.
- Price history: $1.00 per 1,000. One car that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a car is free.

Store page, reviews and full documentation: [Netherlands Car Listings Scraper](https://apify.com/carabain-digital/netherlands-cars)

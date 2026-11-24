---
name: germany-cars
description: "Search mobile.de, AutoScout24, Kleinanzeigen and AutoUncle at once for used and new cars in Germany. One search, one dataset: merged, deduplicated across the sites, newest first. Filter by make, model, year, price, mileage, fuel, location. Optional full details and price history. JSON, CSV or Excel. Runs the Apify Actor carabain-digital/germany-cars. Use for tasks like: Used BMW X1 under 30,000 EUR from mobile.de and AutoScout24; Electric cars in Germany with price history."
---

# Germany Cars Scraper

Search mobile.de, AutoScout24, Kleinanzeigen and AutoUncle at once for used and new cars in Germany. One search, one dataset: merged, deduplicated across the sites, newest first. Filter by make, model, year, price, mileage, fuel, location. Optional full details and price history. JSON, CSV or Excel.

Runs on Apify as the Actor [`carabain-digital/germany-cars`](https://apify.com/carabain-digital/germany-cars?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 8,333 car listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~germany-cars/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~germany-cars/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"make": "BMW", "model": "X1", "condition": "used", "max_price": 30000, "limit": 200}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/germany-cars").call(run_input={"make": "BMW", "model": "X1", "condition": "used", "max_price": 30000, "limit": 200})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/germany-cars`.

## Input

| Field | Type | Description |
|---|---|---|
| `make` | string | Car make, sent to every site. Example: `BMW`, `Volkswagen`, `Mercedes-Benz`, `Tesla`. Leave empty for all makes. |
| `model` | string | Model of that make, sent to every site. Example: `Golf`, `X1`, `C-Klasse`, `Model 3`, `3er` (BMW 3 Series: all 316 ... 340). Needs a make. A site that does not know the model is skipped. |
| `min_year` | integer | Earliest year of first registration. Example: `2018`. |
| `max_year` | integer | Latest year of first registration. Example: `2022`. |
| `min_price` | integer | Lowest asking price in euro. Example: `5000`. |
| `max_price` | integer | Highest asking price in euro. Example: `25000`. |
| `max_mileage` | integer | Highest mileage in km. Example: `100000`. |
| `fuel_type` | string | Fuel type. Example: `electric`. `plugin_hybrid`: mobile.de filters plug-ins exactly, AutoScout24 and Kleinanzeigen return all hybrids. Values: `any`, `petrol`, `diesel`, `electric`, `hybrid`, `plugin_hybrid`, `lpg`, `cng`, `hydrogen`, `ethanol`, `other`. Default: `"any"`. |
| `transmission` | string | Gearbox. Example: `automatic`. Values: `any`, `manual`, `automatic`. Default: `"any"`. |
| `body_type` | string | mobile.de category. Pickups are in the SUV category and minivans in Van/Minibus. Example: `suv`. Kleinanzeigen maps `pickup` to SUV and `van` to its bus category. Values: `any`, `sedan`, `hatchback`, `wagon`, `suv`, `pickup`, `coupe`, `convertible`, `van`, `minivan`, `other`. Default: `"any"`. |
| `condition` | string | New or used cars. Example: `used`. Kleinanzeigen has no such filter (it is almost all used): `used` is ignored there, `new` skips Kleinanzeigen. Values: `any`, `used`, `new`. Default: `"any"`. |
| `seller_type` | string | Dealers or private sellers. Example: `dealer`. Values: `any`, `dealer`, `private`. Default: `"any"`. |
| `location` | string | German postcode or city to search around, sent to every site. Example: `10115`, `München`, `Köln`. Several places separated by commas run as separate searches. Leave empty for all of Germany. |
| `radius_km` | integer | Distance around the location, in km. Example: `50`. Only used with a location. Kleinanzeigen takes at most 200 km. Default: `50`. |
| `sites` | array | Which sites to search. Default: mobile.de, AutoScout24 and Kleinanzeigen. AutoUncle collects ads from other sites, so it adds mostly duplicates (plus its price rating and market value); select it on purpose. Example: `["mobilede"]`. Default: `["mobilede", "autoscout24", "kleinanzeigen"]`. |
| `limit` | integer | Cap on the merged dataset, newest first; every site gets a fair share. Example: `500`. Default: `100`. |
| `limit_per_site` | integer | Optional cap per site, to keep the mix balanced. Default: Max listings. Example: `250`. |
| `includeDetails` | boolean | Also open each listing on its site: description, equipment, colour, doors, seats, previous owners, coordinates, listing date, all photos and seller phones, plus mobile.de's attributes, AutoScout24's and AutoUncle's market price and Kleinanzeigen's HU date and emission class. Adds $0.95 per 1,000 listings that were actually enriched. Default: `false`. |
| `trackHistory` | boolean | Keep the price of every listing in a named store in your own Apify account and add `first_seen_at`, `previous_price`, `price_change`, `price_change_pct`, `status_changed` and `price_history` to every car you scraped before. Shared with our single-site car Actors. Adds $1.00 per 1,000 listings that already had history (the first run only starts the series). Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `golf-berlin`. Default: `"cars-history"`. |

## Examples

**Used BMW X1 under 30,000 EUR from mobile.de and AutoScout24**: The newest used BMW X1 listings under 30,000 euro in Germany from mobile.de, AutoScout24 and Kleinanzeigen in one deduplicated list: price, price rating, mileage, year, fuel, city and dealer.

```json
{
  "make": "BMW",
  "model": "X1",
  "condition": "used",
  "max_price": 30000,
  "limit": 200
}
```

**Electric cars in Germany with price history**: Track used electric cars on mobile.de, AutoScout24 and Kleinanzeigen over time: asking price, price drops since your last run, mileage, year and dealer. Schedule it daily for a price-drop feed.

```json
{
  "fuel_type": "electric",
  "condition": "used",
  "limit": 500,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `source`, `make`, `model`, `year`, `mileage`, `fuel_type`, `price`, `price_rating`, `city`, `seller_type`, `also_on`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `<source>:<site id>` (mobilede:446230057, autoscout24:<uuid>, kleinanzeigen:<ad id>, autouncle:de:<id>) |
| `source` | string | Site the item was taken from: mobilede, autoscout24, kleinanzeigen or autouncle |
| `url` | string | Public listing URL on suchen.mobile.de |
| `title` | string | Listing title (make, model and the seller's model text) |
| `make` | string | Make, e.g. BMW |
| `model` | string | Model as mobile.de classifies it, e.g. 320 |
| `variant` | string | The seller's model description, e.g. 320d xDrive M Sport |
| `condition` | string | used, new or demo |
| `price` | integer or number | Price in EUR (gross, incl. VAT) |
| `price_currency` | string | ISO currency code (EUR) |
| `price_type` | string | fixed, negotiable (VB) or on_request |
| `mileage` | integer or number | Mileage |
| `mileage_unit` | string | km |
| `year` | integer | Year of first registration (new cars: model year) |
| `first_registration` | string | First registration, YYYY-MM |
| `fuel_type` | string | petrol, diesel, electric, hybrid, plugin_hybrid, lpg, cng, hydrogen, ethanol or other |
| `transmission` | string | manual or automatic |
| `body_type` | string | sedan, hatchback (Kleinwagen), wagon, suv (incl. pickups), coupe, convertible, van or other |
| `power_kw` | integer | Power in kW |
| `power_hp` | integer | Power in PS (hp) |
| `engine_cc` | integer or number | Engine displacement in cm3 |
| `color` | string | Colour (details add-on: with finish, e.g. Blau Metallic) |
| `doors` | integer | Number of doors (4/5 -> 5) |
| `seats` | integer | Number of seats |
| `vin` | string | VIN. Neither mobile.de nor AutoScout24 publish it: always null |
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
| `listed_at` | string | When the listing was published (mobile.de: last renewed), ISO 8601. AutoScout24 search results have none; the Full details add-on fills it |
| `image_urls` | array | Photo URLs (1024 px) |
| `description` | string | Details add-on: the seller's description text (German) |
| `status` | string | available |
| `category` | string | mobile.de category label, e.g. SUV/Geländewagen/Pickup |
| `price_net` | number | Net price for dealers that show VAT |
| `vat_rate` | number | VAT rate in percent (dealers) |
| `price_rating` | string | The site's own price rating vs the market, in its own words: mobile.de very_good_price, good_price, reasonable_price, increased_price, high_price, no_rating; AutoScout24 very_good, good, fair, increased, high (null when unrated) |
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
| `seller_contact_name` | string | Dealer contact person |
| `online_purchase` | boolean | True when the car can be bought online through AutoScout24 (smyle) |
| `equipment` | array | Details add-on: equipment and options, in the site's language (e.g. Klimaanlage, Navigationssystem) |
| `gears` | integer | Details add-on: number of gears |
| `cylinders` | integer | Details add-on: number of cylinders |
| `drive_train` | string | Details add-on: drive train (front, rear, 4WD) |
| `weight_kg` | integer | Details add-on: empty weight in kg |
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
| `also_on` | array | Other sites that carry the same car (same make, first-registration year, mileage and price); empty when only one does |
| `promoted` | boolean | Paid top ad (no date on the card, bumped to the top) |
| `reserved` | boolean | Marked as reserved |
| `energy_label` | string | Energy label (NL) |
| `car_pass_url` | string | Not used on this site (always null) |
| `fuel_type_original` | string | Fuel as the site writes it |
| `body_type_original` | string | Body type as the site writes it |
| `license_plate` | string | Details add-on: licence plate (NL) |
| `apk_until` | string | Details add-on: HU (TUeV) expiry as the site writes it, e.g. 'Mai 2028' |
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
| `subtitle` | string | The ad's own headline under the title, e.g. `1.0TSI 85KW DSG LED GRA PDC SHZ` (dealer wording, in the site's language) |
| `price_rating_score` | integer | AutoUncle's price rating as a number: 5 (Superpreis) ... 1 (Teuer), the stars on the site |
| `market_price` | integer | AutoUncle's market value for comparable cars, in price_currency. List results: price + price_below_market (only rated 3-5 stars); Full details add-on: the exact value for every rated car |
| `price_below_market` | integer | How far the price is under AutoUncle's market value (rated 3-5 stars only; null above market) |
| `listing_price_change_pct` | number | Price change since the car was listed, in percent (negative = cheaper), as AutoUncle shows it |
| `days_listed` | integer | Days the car has been advertised (0 = today) |
| `electric_range_km` | integer | Range of an electric car (WLTP) as shown on the card |
| `source_site` | string | Where the ad lives: the dealer, or a marketplace such as Kleinanzeigen, AutoScout24, Pkw.de, Marktplaats |

## Price

Pay per result: $0.60 per 1,000 car listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $0.95 per 1,000. One car listing enriched from its page on mobile.de, AutoScout24, Kleinanzeigen or AutoUncle: description, equipment, colour, doors, seats, previous owners, coordinates, listing date, all photos and seller phones, plus each site's extras (mobile.de attributes, market price, HU date).
- Price history: $1.00 per 1,000. One car listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [Germany Cars Scraper](https://apify.com/carabain-digital/germany-cars?fpr=4ijnom)

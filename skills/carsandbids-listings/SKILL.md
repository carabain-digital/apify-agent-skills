---
name: carsandbids-listings
description: "Scrape live and past car auctions from Cars & Bids (carsandbids.com), the US enthusiast car auction site: current bid or sold price, reserve status, bids, end time, mileage, location, photos, VIN. Filter by make, model, year, price, mileage. Add-ons: full details, price history. Runs the Apify Actor carabain-digital/carsandbids-listings. Use for tasks like: Live Cars & Bids auctions: current bids and reserve status; Sold Porsche 911 prices on Cars & Bids."
---

# Cars & Bids Auction Scraper

Scrape live and past car auctions from Cars & Bids (carsandbids.com), the US enthusiast car auction site: current bid or sold price, reserve status, bids, end time, mileage, location, photos, VIN. Filter by make, model, year, price, mileage. Add-ons: full details, price history.

Runs on Apify as the Actor [`carabain-digital/carsandbids-listings`](https://apify.com/carabain-digital/carsandbids-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 1,052 car auctions with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~carsandbids-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~carsandbids-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"auction_status": "live", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/carsandbids-listings").call(run_input={"auction_status": "live", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/carsandbids-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `auction_status` | string | Which auctions to scrape: `live`, `closed` (all past results), `sold`, `reserve_not_met`, or `any` (live first, then past). Example: `sold` for the prices cars actually fetched. Values: `live`, `closed`, `sold`, `reserve_not_met`, `any`. Default: `"live"`. |
| `make` | string | Car make as in the auction title. Example: `Porsche`, `Ford`, `Mercedes-Benz`, `Land Rover`. |
| `model` | string | Model of that make (needs Make). Matched on the title. Example: `911`, `Bronco`, `S2000`. |
| `min_year` | integer | Oldest model year. Example: `2018`. |
| `max_year` | integer | Newest model year. Example: `2023`. |
| `min_price` | integer | Lowest current bid (live) or sold price / final bid (past), in US dollars. Applied after fetching, so a narrow price range scans more auctions. Example: `10000`. |
| `max_price` | integer | Highest current bid (live) or sold price / final bid (past), in US dollars. Example: `25000`. |
| `min_mileage` | integer | Lowest odometer reading in miles. Example: `1000`. |
| `max_mileage` | integer | Highest odometer reading in miles. Example: `60000`. |
| `transmission` | string | Transmission. Example: `manual`. Values: `any`, `automatic`, `manual`. Default: `"any"`. |
| `body_type` | string | Body style. Example: `coupe`. Values: `any`, `coupe`, `convertible`, `hatchback`, `sedan`, `suv`, `pickup`, `wagon`, `van`, `minivan`. Default: `"any"`. |
| `sort` | string | Order of the results. Example: `highest_price` with `sold` for the most expensive sales. Values: `newest`, `ending_soon`, `lowest_mileage`, `highest_mileage`, `lowest_price`, `highest_price`. Default: `"newest"`. |
| `start_urls` | array | Pages copied from carsandbids.com; they replace the filters above. Accepts auction pages (`https://carsandbids.com/auctions/rJYOzjOK`), `https://carsandbids.com/auctions` (live), `https://carsandbids.com/past-auctions` and search pages like `https://carsandbids.com/search/porsche%20911`. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many auctions. Example: `100`. Default: `100`. |
| `includeDetails` | boolean | Also open each auction: VIN, make and model, exact mileage, engine, body style, drivetrain, colours, title status, seller type, the seller's write-up (highlights, equipment, flaws, modifications, service history), all photos, number of bids, comments, watchers and views, start time, high bidder and buyer. Adds $1.00 per 1,000 auctions on top of the base price. One extra request per auction (about 1 s per 10 auctions). Default: `false`. |
| `trackHistory` | boolean | Keep the price (bid or sold price) and status of every auction in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history to each item. Run it on a schedule to follow live bids and see when auctions end. Adds $1.00 per 1,000 auctions that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `porsche-911`. Default: `"cars-history"`. |

## Examples

**Live Cars & Bids auctions: current bids and reserve status**: Scrape the newest live auctions on Cars & Bids with the current high bid, no-reserve flag, end time, mileage, transmission and location. Run it on a schedule to watch the market for enthusiast cars. Export to JSON, CSV or Excel.

```json
{
  "auction_status": "live",
  "limit": 100
}
```

**Sold Porsche 911 prices on Cars & Bids**: Scrape past Cars & Bids auctions of the Porsche 911 that sold: final price, mileage, transmission, location, end date and whether the reserve was met. Add the full details for VIN, bid count and the seller's write-up. Ideal for valuation and price research.

```json
{
  "auction_status": "sold",
  "make": "Porsche",
  "model": "911",
  "limit": 200
}
```

## Output

One dataset item per result. Main fields: `auction_end`, `title`, `price`, `status`, `mileage`, `city`, `region`, `no_reserve`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable auction ID, `carsandbids:<auction id>` |
| `source` | string | Short site name (carsandbids) |
| `url` | string | Public auction page on carsandbids.com |
| `title` | string | Auction title, e.g. 2019 Ford F-150 Raptor SuperCrew |
| `make` | string | Make (from the title; the real make with the details add-on) |
| `model` | string | Model and trim as in the title, e.g. F-150 Raptor SuperCrew (just the model with the details add-on) |
| `variant` | string | Details add-on: trim words after the model, e.g. SuperCrew |
| `condition` | string | Always used (Cars & Bids sells used cars) |
| `price` | number | Current high bid for a live auction; the sold price for a sold car; the final high bid when the reserve was not met. USD |
| `price_currency` | string | USD |
| `price_type` | string | Always auction |
| `mileage` | integer | Odometer reading as listed (exact with the details add-on) |
| `mileage_unit` | string | mi (miles) |
| `year` | integer | Model year |
| `first_registration` | string | Always null: US listings only have the model year |
| `fuel_type` | string | Details add-on: petrol, diesel, electric, hybrid or plugin_hybrid, guessed from the engine text |
| `transmission` | string | manual or automatic |
| `body_type` | string | Details add-on: coupe, convertible, hatchback, sedan, suv, pickup, van or wagon |
| `power_kw` | integer | Details add-on: power in kW, when the engine text states hp |
| `power_hp` | integer | Details add-on: power in hp, when the engine text states it |
| `engine_cc` | integer | Details add-on: engine displacement in cc (from e.g. 3.5L) |
| `color` | string | Details add-on: exterior colour |
| `doors` | integer | Always null: not published |
| `seats` | integer | Always null: not published |
| `vin` | string | Details add-on: vehicle identification number |
| `seller_type` | string | Details add-on: private or dealer |
| `seller_name` | string | Seller's Cars & Bids username |
| `seller_phone` | string | Always null: not published |
| `seller_url` | string | Seller's profile page on Cars & Bids |
| `address` | string | Location as shown, e.g. Raleigh, NC 27614 |
| `city` | string | City |
| `region` | string | State or province, e.g. NC |
| `postcode` | string | ZIP code (US listings) |
| `country` | string | US or CA |
| `lat` | number | Always null: not published |
| `lon` | number | Always null: not published |
| `listed_at` | string | Details add-on: date the auction started (YYYY-MM-DD) |
| `image_urls` | array | Photo URLs (one without details, the whole gallery with the details add-on) |
| `description` | string | Details add-on: the seller's highlights write-up |
| `subtitle` | string | The auction's tagline, e.g. 450-hp Twin-Turbo V6, 4WD, Forged Wheels |
| `status` | string | live, sold, sold_after (sold after the auction at a negotiated price), reserve_not_met, last_second_bids, canceled or no_bids |
| `sold` | boolean | True when the car sold (sold or sold_after), false for reserve not met, null while live |
| `current_bid` | number | Highest bid in USD (the sold price when sold) |
| `sold_price` | number | Sale price in USD; null unless the car sold |
| `bid_count` | integer | Details add-on: number of bids |
| `auction_end` | string | When the auction ends or ended, UTC ISO 8601 |
| `auction_start` | string | Details add-on: when the auction started, UTC ISO 8601 |
| `no_reserve` | boolean | True for a no-reserve auction (the car sells to the highest bidder) |
| `reserve_met` | boolean | True when the reserve is met (sold, or no reserve); false when it was not; null while live with a reserve |
| `mileage_text` | string | Mileage as the seller wrote it, e.g. 86,200 Miles Shown - TMU (true mileage unknown) |
| `featured` | boolean | True for featured auctions |
| `has_inspection` | boolean | True when the car has a Cars & Bids inspection |
| `comment_count` | integer | Details add-on: number of comments |
| `watcher_count` | integer | Details add-on: number of watchers |
| `view_count` | integer | Details add-on: number of page views |
| `high_bidder` | string | Details add-on: username of the current high bidder |
| `buyer` | string | Details add-on: username of the buyer when sold |
| `title_status` | string | Details add-on: title status, e.g. Clean (NC) |
| `drivetrain` | string | Details add-on: Front-wheel drive, Rear-wheel drive or 4WD/AWD |
| `engine` | string | Details add-on: engine, e.g. 3.5L Turbocharged V6 |
| `transmission_text` | string | Details add-on: transmission, e.g. 10-Speed Automatic |
| `interior_color` | string | Details add-on: interior colour |
| `highlights` | array | Details add-on: the seller's bullet points |
| `equipment` | string | Details add-on: the seller's equipment write-up |
| `known_flaws` | string | Details add-on: flaws the seller reports |
| `modifications` | string | Details add-on: modifications the seller reports |
| `service_history` | string | Details add-on: recent service history write-up |
| `ownership_history` | string | Details add-on: ownership history write-up |
| `seller_notes` | string | Details add-on: extra seller notes |
| `history_report_url` | string | Details add-on: Carfax or AutoCheck report link, when the seller provided one |
| `details_fetched` | boolean | True when the details add-on enriched this item |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: true when the auction status changed since the previous run (e.g. live to sold) |
| `price_history` | array | Price history add-on: price points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $4.75 per 1,000 car auctions on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One auction enriched with its detail data: VIN, exact mileage, engine, body style, drivetrain, colours, title status, seller write-up, all photos, bid, comment, watcher and view counts, start time, high bidder and buyer.
- Price history: $1.00 per 1,000. One auction that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of an auction is free.

Store page, reviews and full documentation: [Cars & Bids Auction Scraper](https://apify.com/carabain-digital/carsandbids-listings?fpr=4ijnom)

---
name: bringatrailer-listings
description: "Scrape Bring a Trailer (bringatrailer.com) collector-car auctions: live bids and completed results with sold prices, reserve status, year, make, model, mileage. Filter by make, model, year, price. Newest first. Add-ons: VIN, bids, seller, photos; price history. Runs the Apify Actor carabain-digital/bringatrailer-listings. Use for tasks like: Sold Porsche 911 prices on Bring a Trailer; Live No Reserve car auctions on Bring a Trailer."
---

# Bring a Trailer Auctions Scraper

Scrape Bring a Trailer (bringatrailer.com) collector-car auctions: live bids and completed results with sold prices, reserve status, year, make, model, mileage. Filter by make, model, year, price. Newest first. Add-ons: VIN, bids, seller, photos; price history.

Runs on Apify as the Actor [`carabain-digital/bringatrailer-listings`](https://apify.com/carabain-digital/bringatrailer-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 1,176 auctions with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~bringatrailer-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~bringatrailer-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"auction_status": "completed", "make": "Porsche", "model": "911", "min_year": 1990, "limit": 200}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/bringatrailer-listings").call(run_input={"auction_status": "completed", "make": "Porsche", "model": "911", "min_year": 1990, "limit": 200})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/bringatrailer-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `auction_status` | string | Which auctions to scrape: `completed` (sold prices and reserve-not-met results, most recently ended first), `live` (open auctions, latest-ending first) or `all`. Example: `live`. Values: `completed`, `live`, `all`. Default: `"completed"`. |
| `auction_result` | string | Keep only auctions that were sold, or only those that ended below the reserve. Live auctions are skipped when you pick one. Example: `sold`. Values: `any`, `sold`, `unsold`. Default: `"any"`. |
| `make` | string | Make to search in the auction titles. Example: `Porsche`, `Mercedes-Benz`, `Land Rover`. |
| `model` | string | Model of that make (needs Make). Example: `911`, `300SL`, `Defender`. |
| `min_year` | integer | Oldest model year. Example: `1990`. |
| `max_year` | integer | Newest model year. Example: `1999`. |
| `min_price` | integer | Lowest current bid, sold price or high bid in US dollars. Example: `20000`. |
| `max_price` | integer | Highest current bid, sold price or high bid in US dollars. Example: `50000`. |
| `max_mileage` | integer | Highest odometer reading in miles. The list only knows the mileage the title mentions (`36k-Mile`); auctions without it are kept. Example: `60000`. |
| `no_reserve_only` | boolean | Keep only No Reserve auctions (they always sell). Example: `true`. Default: `false`. |
| `includeNonVehicles` | boolean | BaT also auctions parts, wheels, memorabilia, motorcycles, boats and RVs. They are skipped by default (recognised from the title, or the category on live auctions). Switch on to get everything. Example: `false`. Default: `false`. |
| `start_urls` | array | Individual auction pages copied from bringatrailer.com; they replace the filters above and come with all details. Example: `https://bringatrailer.com/listing/1997-jeep-grand-cherokee-23/`. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many auctions (newest first). BaT serves at most 10,000 completed auctions per search; narrow the year range for more. Example: `100`. Default: `100`. |
| `includeDetails` | boolean | Also open each auction page: VIN, exact mileage, engine, transmission, colour, seller, location, lot number, number of bids, buyer, views, watchers, all photos and the full summary. Adds $1.00 per 1,000 auctions on top of the base price. One extra request per auction (about 1.5 s per 10). Default: `false`. |
| `trackHistory` | boolean | Keep the price of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our car Actors use `cars-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `camry-nyc`. Default: `"cars-history"`. |

## Examples

**Sold Porsche 911 prices on Bring a Trailer**: Scrape the most recent completed Porsche 911 auctions from Bring a Trailer (1990 and newer): sold price or high bid, reserve status, year, model and mileage. Export to JSON, CSV or Excel to build price comps.

```json
{
  "auction_status": "completed",
  "make": "Porsche",
  "model": "911",
  "min_year": 1990,
  "limit": 200
}
```

**Live No Reserve car auctions on Bring a Trailer**: Scrape the open Bring a Trailer auctions that have no reserve and track their price over time: current bid, end time, year, make and model. Run it daily to see where bids end up.

```json
{
  "auction_status": "live",
  "no_reserve_only": true,
  "limit": 200,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `auction_end_at`, `title`, `status`, `price`, `no_reserve`, `mileage`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `bringatrailer:<BaT post id>` |
| `source` | string | Short site name (bringatrailer) |
| `url` | string | Public auction page on bringatrailer.com |
| `title` | string | Auction title, e.g. 7k-Mile 2021 Porsche 911 Carrera 4S Cabriolet |
| `make` | string | Make, split from the title (e.g. Porsche, Mercedes-Benz) |
| `model` | string | Model and trim as in the title, after the make |
| `variant` | string | Trim, e.g. LX AWD |
| `condition` | string | Always `used` (BaT sells pre-owned and collector cars) |
| `price` | number | Current bid (live), sold price (sold) or high bid (reserve not met), in USD; null when a live auction has no bid yet |
| `price_currency` | string | USD |
| `price_type` | string | `auction` when there is a bid, else `on_request` |
| `mileage` | integer | Odometer. From the title (`36k-Mile`, rounded to thousands) or, with Full details, the listing's mileage (`mileage_approximate` says which) |
| `mileage_unit` | string | mi (miles) |
| `year` | integer | Model year |
| `first_registration` | string | Always null: US listings only have the model year |
| `fuel_type` | string | petrol, diesel, electric, hybrid, plugin_hybrid, ethanol (flex fuel), cng, hydrogen or other |
| `transmission` | string | manual or automatic |
| `body_type` | string | sedan, hatchback, wagon, suv, coupe, convertible, van, pickup, minivan or other |
| `power_kw` | integer | Engine power in kW (from hp) |
| `power_hp` | integer | Engine power in hp, when CarGurus knows it |
| `engine_cc` | integer | Engine displacement in cc |
| `color` | string | Exterior colour as the dealer names it |
| `doors` | integer | Number of doors |
| `seats` | integer | Always null: not published |
| `vin` | string | Chassis number / VIN (Full details add-on) |
| `seller_type` | string | dealer or private (Full details add-on) |
| `seller_name` | string | Seller's BaT username (Full details add-on) |
| `seller_phone` | string | Dealer phone number |
| `seller_url` | string | Details add-on: dealer website |
| `address` | string | Dealer address (city and state; full street address with the details add-on) |
| `city` | string | City of the car (Full details add-on) |
| `postcode` | string | Dealer ZIP code |
| `region` | string | State or province (Full details add-on) |
| `country` | string | ISO country code of the car, e.g. US, CA, DE |
| `lat` | number | Details add-on: dealer latitude |
| `lon` | number | Details add-on: dealer longitude |
| `listed_at` | string | Always null: BaT does not publish the listing start |
| `image_urls` | array | Photo URLs: the thumbnail, or every photo with Full details |
| `description` | string | The auction summary (the full text of the listing's intro with Full details) |
| `status` | string | `live`, `sold`, `reserve_not_met` (ended without meeting the reserve) or `ended` |
| `auction_end_at` | string | When the auction ends or ended, UTC ISO |
| `current_bid` | number | Current bid in USD while the auction is live |
| `sold_price` | number | Final price in USD of a sold auction |
| `high_bid` | number | Highest bid in USD of an auction that ended below its reserve |
| `sold` | boolean | true when sold, false when the reserve was not met, null while live |
| `no_reserve` | boolean | true for No Reserve auctions |
| `bid_count` | integer | Number of bids (Full details add-on) |
| `comment_count` | integer | Number of comments (Full details add-on) |
| `view_count` | integer | Page views (Full details add-on) |
| `watcher_count` | integer | Users watching the auction (Full details add-on) |
| `lot_number` | integer | BaT lot number (Full details add-on) |
| `seller_username` | string | Seller's BaT username (Full details add-on) |
| `winning_bidder` | string | Buyer's username of a sold auction (Full details add-on) |
| `premium` | boolean | BaT Premium listing |
| `bat_alumni` | boolean | Car that was listed on BaT before |
| `categories` | array | Category ids on the live list; category slugs (german, convertible, truck-4x4 ...) with Full details; empty on completed auctions otherwise |
| `mileage_approximate` | boolean | true when the mileage is rounded (`36k`) |
| `engine` | string | Engine as the listing words it, e.g. 3.6-Liter Flat-Six (Full details add-on) |
| `transmission_text` | string | Transmission as the listing words it (Full details add-on) |
| `exterior_color` | string | Paint colour (Full details add-on) |
| `details_fetched` | boolean | true when the Full details add-on enriched this item |
| `scraped_at` | string | When the item was scraped, UTC ISO |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: always false for CarGurus (no status field) |
| `price_history` | array | Price history add-on: price points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $4.25 per 1,000 auctions on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One auction enriched with its listing page: VIN, exact mileage, engine, transmission, colour, seller, location, lot number, number of bids, buyer, views, watchers, all photos and the full summary.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [Bring a Trailer Auctions Scraper](https://apify.com/carabain-digital/bringatrailer-listings?fpr=4ijnom)

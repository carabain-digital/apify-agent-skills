---
name: pap-listings
description: "Scrape private-seller property listings from PAP (pap.fr, De Particulier à Particulier): apartments, houses, land and parking for sale and for rent, no agencies. Filter by city, price, rooms and area. Price, DPE, photos. Add-on: full description, street and publication date. Runs the Apify Actor carabain-digital/pap-listings. Use for tasks like: Apartments for sale by owner in Paris: newest on PAP; Rentals from owners in Lyon with price history on PAP."
---

# PAP.fr Scraper - French Private-Seller Listings

Scrape private-seller property listings from PAP (pap.fr, De Particulier à Particulier): apartments, houses, land and parking for sale and for rent, no agencies. Filter by city, price, rooms and area. Price, DPE, photos. Add-on: full description, street and publication date.

Runs on Apify as the Actor [`carabain-digital/pap-listings`](https://apify.com/carabain-digital/pap-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 4,545 listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~pap-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~pap-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"location": "Paris", "listing_type": "sale", "property_type": "apartment", "limit": 100, "includeDetails": false}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/pap-listings").call(run_input={"location": "Paris", "listing_type": "sale", "property_type": "apartment", "limit": 100, "includeDetails": false})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/pap-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `location` | string | French city, arrondissement, department or postcode (required: PAP only searches inside a place). Several values separated by commas run one search each. Example: `Lyon`, `Paris 15e`, `33000` or `Gironde`. Ignored when you give search URLs. |
| `listing_type` | string | `sale` for properties to buy, `rent` for rentals. Example: `rent`. Values: `sale`, `rent`. Default: `"sale"`. |
| `property_type` | string | Type of property. `any` = every type PAP lists. `commercial` = offices (bureaux). Example: `apartment`. Values: `any`, `house`, `apartment`, `land`, `commercial`, `parking`. Default: `"any"`. |
| `min_price` | integer | Lowest price in euro (rent: per month, charges included as on the site). Applied exactly to every listing. Example: `200000`, for rent `600`. |
| `max_price` | integer | Highest price in euro (rent: per month). Applied exactly to every listing. Example: `450000`, for rent `1200`. |
| `min_rooms` | integer | Minimum number of rooms (pièces, a studio counts 1). Listings without a room count are left out. Example: `3`. |
| `min_living_area_m2` | integer | Minimum living area in square metres. Listings without a known area are left out. Example: `50`. |
| `start_urls` | array | Search result pages copied from pap.fr; they replace the filters above. Example: `https://www.pap.fr/annonce/vente-appartements-paris-75-g439-jusqu-a-500000-euros`. Filters in the URL are kept (PAP applies them with a margin); the page number is ignored. Leave empty to use the filters above. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (most recent first). Example: `100`. PAP shows at most 25 pages (about 300-375 listings) per search: use arrondissements, price ranges or several search URLs to go deeper. Default: `100`. |
| `includeDetails` | boolean | Also open each listing's page: full description, all photos in full size, street, publication date and plot area. Adds $2.00 per 1,000 listings on top of the base price. One extra request per listing (about 1 s each), so runs take longer. Default: `false`. |
| `trackHistory` | boolean | Keep the price and status of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our real-estate Actors use `realestate-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `gent-apartments`. Default: `"realestate-history"`. |

## Examples

**Apartments for sale by owner in Paris: newest on PAP**: Scrape the newest apartments for sale directly from their owners in Paris on PAP (pap.fr): price, arrondissement, rooms, living area, bedrooms and DPE label. No agency fees. Export to JSON, CSV or Excel.

```json
{
  "location": "Paris",
  "listing_type": "sale",
  "property_type": "apartment",
  "limit": 100,
  "includeDetails": false
}
```

**Rentals from owners in Lyon with price history on PAP**: Scrape rentals posted by private landlords in Lyon on PAP (pap.fr) and track their rent over time: monthly rent, rooms, living area and DPE. Run it daily to catch rent changes and new listings.

```json
{
  "location": "Lyon",
  "listing_type": "rent",
  "max_price": 1500,
  "limit": 200,
  "includeDetails": false,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `title`, `city`, `price`, `living_area_m2`, `rooms`, `status`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `<source>:<site id>` |
| `source` | string | Short site name |
| `url` | string | Public listing URL on the site |
| `title` | string | Listing title as the agent wrote it (not every listing has one) |
| `listing_type` | string | sale or rent |
| `property_type` | string | house, apartment, land, commercial, parking, room or other |
| `property_subtype` | string | The site's own type: appartement, maison, terrain, garage-parking, bureaux, immeuble ... |
| `status` | string | available, under_offer, sold or rented |
| `price` | number | Asking price in EUR (rent per month, charges included, as PAP shows it). Null for listings without a price |
| `price_currency` | string | ISO currency code (EUR) |
| `price_period` | string | month for rentals, null for sales |
| `price_type` | string | asking, on_request, auction (public/online sale), life_annuity or from (new-build project, minimum price) |
| `address` | string | Postcode and city; with the Full details add-on also the street |
| `street` | string | Street (and sometimes a building name). Only with the Full details add-on; PAP hides the house number |
| `house_number` | string | Always null: PAP does not show the house number |
| `postcode` | string | Postcode. Some cards of the search pages omit it (filled by the Full details add-on) |
| `city` | string | City / municipality |
| `region` | string | Always null: PAP does not give one |
| `country` | string | ISO country code (BE) |
| `lat` | number | Always null: PAP does not publish coordinates in its listing pages |
| `lon` | number | Always null, see lat |
| `living_area_m2` | number | Living area in m2 |
| `plot_area_m2` | number | Plot area in m2 |
| `rooms` | integer | Total number of rooms, when the site gives it |
| `bedrooms` | integer | Number of bedrooms |
| `bathrooms` | integer | Always null: PAP does not give a bathroom count in structured form |
| `energy_label` | string | DPE energy label (A to G) from the search card |
| `construction_year` | integer | Always null: PAP does not give one in structured form |
| `listed_at` | string | Publication date of the ad (date only, YYYY-MM-DD; PAP renews ads, so it is the latest publication). Only with the Full details add-on |
| `updated_at` | string | When the listing was last changed on the site, ISO 8601 |
| `agent_name` | string | Always null: PAP sellers are private persons |
| `agent_phone` | string | Always null: PAP shows the phone number only after a click on the site |
| `agent_url` | string | Always null: PAP sellers are private persons |
| `image_urls` | array | Photo URLs (small webp from the search card; full-size jpg with the Full details add-on) |
| `description` | string | Full description written by the seller. Only with the Full details add-on |
| `old_price` | number | Always null on PAP (use the Price history add-on) |
| `monthly_costs` | number | Always null: PAP rents are shown charges included, without a separate amount in the data |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: true when the status changed since the previous run |
| `price_history` | array | Price history add-on: price and status points over your runs, `[{at, price, status}]`, last 20 |

## Price

Pay per result: $1.10 per 1,000 listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $2.00 per 1,000. One listing enriched with its page: full description, all photos in full size, street, publication date and plot area.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [PAP.fr Scraper - French Private-Seller Listings](https://apify.com/carabain-digital/pap-listings?fpr=4ijnom)

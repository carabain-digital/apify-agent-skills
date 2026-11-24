---
name: pararius-listings
description: "Scrape Dutch rental and sale listings from Pararius (pararius.nl and .com): price, address, living area, rooms, energy label, agent and photos. Filter by city, price, type, rooms and size, newest first. Optional full details and price history. Runs the Apify Actor carabain-digital/pararius-listings. Use for tasks like: Newest Amsterdam rentals under 2000 euro; Utrecht apartments for sale on Pararius."
---

# Pararius Listings Scraper

Scrape Dutch rental and sale listings from Pararius (pararius.nl and .com): price, address, living area, rooms, energy label, agent and photos. Filter by city, price, type, rooms and size, newest first. Optional full details and price history.

Runs on Apify as the Actor [`carabain-digital/pararius-listings`](https://apify.com/carabain-digital/pararius-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 5,000 listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~pararius-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~pararius-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"location": "Amsterdam", "listing_type": "rent", "max_price": 2000, "limit": 50}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/pararius-listings").call(run_input={"location": "Amsterdam", "listing_type": "rent", "max_price": 2000, "limit": 50})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/pararius-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `location` | string | City or place name as Pararius spells it. Example: `Amsterdam`, `Den Haag`, `Utrecht`. Leave empty for all of the Netherlands. Ignored when you give Start URLs. |
| `listing_type` | string | Search homes `rent` or `sale`. Example: `rent`. Ignored when you give Start URLs. Values: `sale`, `rent`. Default: `"rent"`. |
| `min_price` | integer | Lowest asking price in euro (rent per month for rentals). Example: `1000` (rent) or `250000` (sale). |
| `max_price` | integer | Highest asking price in euro (rent per month for rentals). Example: `2000` (rent) or `500000` (sale). |
| `property_type` | string | Only this type of home. Example: `apartment`. `any` returns every type. Values: `any`, `apartment`, `house`, `studio`, `room`. Default: `"any"`. |
| `min_rooms` | integer | Only homes with at least this many rooms (living room included). Example: `3`. |
| `min_living_area_m2` | integer | Only homes with at least this living area in square metres. Example: `60`. |
| `start_urls` | array | Search result URLs copied from pararius.nl (with your own filters). They replace the filter fields above. Example: `https://www.pararius.nl/huurwoningen/amsterdam/1000-2000`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `100`. Up to 10,000, split very large searches by city or price range. Default: `100`. |
| `includeDetails` | boolean | Also open each listing's detail page: full description, all photos, coordinates, date listed, bedrooms, bathrooms, plot area, energy label, build year and the agent phone (sale listings). Adds $3.00 per 1,000 listings on top of the base price. Slower: about 3 seconds per listing. Default: `false`. |
| `trackHistory` | boolean | Keep a price and status history per listing in a named storage in your own account and add `previous_price`, `price_change`, `price_change_pct`, `status_changed` and `price_history` to every listing you have scraped before. Adds $1.00 per 1,000 listings that already had history (the first run only starts the series). Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (a-z, 0-9 and dashes). Use the same name in all our real-estate Actors to share one history. Example: `realestate-history`. Default: `"realestate-history"`. |

## Examples

**Newest Amsterdam rentals under 2000 euro**: Scrape the newest rental listings in Amsterdam up to 2000 euro per month from Pararius: price, address, living area, rooms, furnishing and agent. Schedule it daily to catch new rentals first, and export to JSON, CSV or Excel.

```json
{
  "location": "Amsterdam",
  "listing_type": "rent",
  "max_price": 2000,
  "limit": 50
}
```

**Utrecht apartments for sale on Pararius**: Scrape apartments for sale in Utrecht from Pararius with at least 50 m2 of living area: asking price, address, rooms, build year and agent. Use it for market research or price tracking, and export to JSON, CSV or Excel.

```json
{
  "location": "Utrecht",
  "listing_type": "sale",
  "property_type": "apartment",
  "min_living_area_m2": 50,
  "limit": 50
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `title`, `city`, `price`, `property_type`, `living_area_m2`, `rooms`, `status`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `<source>:<site id>` |
| `source` | string | Short site name |
| `url` | string | Public listing URL |
| `title` | string | Listing title (street and house number, or the site's own title) |
| `listing_type` | string | sale or rent |
| `property_type` | string | apartment, house, studio, room or the site's own type |
| `status` | string | available, under_offer, sold or rented |
| `price` | number | Asking price in EUR (rent per month for rentals). Null when on request |
| `price_currency` | string | ISO currency code |
| `price_period` | string | month for rentals, null for sales |
| `price_type` | string | asking, on_request, or the site's price type (auction ...) |
| `address` | string | Street, house number, postcode and city |
| `street` | string | Street name |
| `house_number` | string | House number with addition. Some rental agents hide it |
| `postcode` | string | Dutch postcode without space, e.g. 1012NJ |
| `city` | string | City |
| `region` | string | Province or region, or null |
| `country` | string | ISO country code (NL) |
| `lat` | number | Latitude. Only with the Full details add-on |
| `lon` | number | Longitude. Only with the Full details add-on |
| `living_area_m2` | number | Living area in m2 |
| `plot_area_m2` | number | Plot area in m2 (houses) |
| `rooms` | number | Number of rooms |
| `bedrooms` | number | Number of bedrooms |
| `bathrooms` | number | Number of bathrooms. Only with the Full details add-on |
| `energy_label` | string | Energy label, e.g. A, B, A+ |
| `construction_year` | number | Year built |
| `listed_at` | string | When the listing was published: ISO 8601 (date only from the detail page on Pararius) |
| `agent_name` | string | Estate agent / landlord |
| `agent_phone` | string | Agent phone number when the site shows it |
| `agent_url` | string | Agent page on the site |
| `image_urls` | array | Photo URLs. All photos with the Full details add-on |
| `description` | string | Full description. Only with the Full details add-on |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `neighbourhood` | string | Neighbourhood (buurt) |
| `furnishing` | string | Furnished, upholstered or shell, as the card says (rentals) |
| `details_fetched` | boolean | True when the Full details add-on enriched this item |
| `first_seen_at` | string | Price history add-on: when your history store first saw this listing |
| `previous_price` | number | Price history add-on: the price at the previous run that saw it |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: change in percent |
| `status_changed` | boolean | Price history add-on: status differs from the previous run |
| `price_history` | array | Price history add-on: last 20 price/status points ({at, price, status}) |

## Price

Pay per result: $1.00 per 1,000 listings on the Apify Free plan, less on paid plans, plus $0.025 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- Full details: $3.00 per 1,000. One listing enriched with its detail page: full description, all photos, coordinates, date listed, bedrooms, bathrooms, plot area, energy label, build year and the agent phone number (sale listings).
- Price history: $1.00 per 1,000. One listing that already had a history in your named store, annotated with previous price, price change and the price/status history.

Store page, reviews and full documentation: [Pararius Listings Scraper](https://apify.com/carabain-digital/pararius-listings?fpr=4ijnom)

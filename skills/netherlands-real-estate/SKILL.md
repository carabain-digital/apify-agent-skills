---
name: netherlands-real-estate
description: "Search Huislijn and Pararius at once for huurwoningen and koopwoningen in the Netherlands. One search, one dataset: merged, deduplicated on postcode and house number, newest first. Optional details and price history. Proxy included. JSON, CSV, Excel. Runs the Apify Actor carabain-digital/netherlands-real-estate. Use for tasks like: Huurwoningen in Utrecht from Huislijn and Pararius; Apartments for sale in the Netherlands with price history."
---

# Netherlands Real Estate Scraper

Search Huislijn and Pararius at once for huurwoningen and koopwoningen in the Netherlands. One search, one dataset: merged, deduplicated on postcode and house number, newest first. Optional details and price history. Proxy included. JSON, CSV, Excel.

Runs on Apify as the Actor [`carabain-digital/netherlands-real-estate`](https://apify.com/carabain-digital/netherlands-real-estate), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 5,000 listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~netherlands-real-estate/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~netherlands-real-estate/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"location": "Utrecht", "listing_type": "rent", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/netherlands-real-estate").call(run_input={"location": "Utrecht", "listing_type": "rent", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/netherlands-real-estate`.

## Input

| Field | Type | Description |
|---|---|---|
| `location` | string | Dutch city or postcode, sent to all selected sites. Several places separated by commas run as separate searches. Example: `Utrecht`, `Amsterdam, Rotterdam` or `3511`. Leave empty for all of the Netherlands. |
| `listing_type` | string | Search homes `rent` or `sale`. Example: `rent`. Values: `sale`, `rent`. Default: `"sale"`. |
| `min_price` | integer | Lowest asking price in euro (rent per month for rentals). Example: `1000` (rent) or `250000` (sale). |
| `max_price` | integer | Highest asking price in euro (rent per month for rentals). Example: `2000` (rent) or `500000` (sale). |
| `property_type` | string | Only this type of property. Example: `apartment`. `any` returns every type. Pararius only knows house, apartment, studio and room; Huislijn has no studio or room. A site that does not know the type is skipped. Values: `any`, `house`, `apartment`, `studio`, `room`, `land`, `commercial`, `parking`. Default: `"any"`. |
| `min_rooms` | integer | Only homes with at least this many rooms (kamers, living room included). Example: `3`. |
| `min_living_area_m2` | integer | Only homes with at least this living area in square metres. Example: `60`. |
| `sites` | array | Which sites to search. Default: Huislijn and Pararius. Example: `["huislijn", "pararius"]`. Default: `["huislijn", "pararius"]`. |
| `limit` | integer | Cap on the merged dataset, newest first. Example: `100`. Default: `100`. |
| `limit_per_site` | integer | Optional cap per site, to keep the mix balanced and browser sites short. Default: Max listings. Example: `25`. |
| `includeDetails` | boolean | Also open each listing's detail page on Huislijn and Pararius: description, coordinates, area, agent and more. Adds $2.00 per 1,000 listings that were actually enriched. Pararius fetches details inside its browser run and only for the listings it fetched (best effort: a bot wall can stop it early, those listings are not charged). Runs take longer. Default: `false`. |
| `trackHistory` | boolean | Keep a price and status history per listing in a named storage in your own account and add `first_seen_at`, `previous_price`, `price_change`, `price_change_pct`, `status_changed` and `price_history` to every listing you have scraped before. The history store is shared with our single-site real-estate Actors. Adds $1.00 per 1,000 listings that already had history (the first run only starts the series). Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (a-z, 0-9 and dashes). Use the same name in all our real-estate Actors to share one history. Example: `realestate-history`. Default: `"realestate-history"`. |

## Examples

**Huurwoningen in Utrecht from Huislijn and Pararius**: The newest rental homes in Utrecht from Huislijn and Pararius in one deduplicated list, with price, address, area and agent.

```json
{
  "location": "Utrecht",
  "listing_type": "rent",
  "limit": 100
}
```

**Apartments for sale in the Netherlands with price history**: The newest apartments for sale in the Netherlands from Huislijn, deduplicated, with price changes since your last run. Schedule it daily for a price-drop feed.

```json
{
  "listing_type": "sale",
  "property_type": "apartment",
  "sites": [
    "huislijn"
  ],
  "limit": 500,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `source`, `title`, `city`, `country`, `price`, `property_type`, `living_area_m2`, `rooms`, `also_on`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `<source>:<site id>` |
| `source` | string | Site the item was taken from: huislijn, pararius |
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
| `house_number` | string | House number with addition. Used for cross-site dedup; some rental agents hide it |
| `postcode` | string | Postcode without space (NL: 1012NJ, BE: 9000). Used for cross-site dedup |
| `city` | string | City |
| `region` | string | Province or region, or null |
| `country` | string | ISO country code of the site (NL or BE) |
| `lat` | number | Latitude. Only with the Full details add-on |
| `lon` | number | Longitude. Only with the Full details add-on |
| `living_area_m2` | number | Living area in m2 |
| `plot_area_m2` | number | Plot area in m2 (houses) |
| `rooms` | number | Number of rooms |
| `bedrooms` | number | Number of bedrooms |
| `bathrooms` | number | Number of bathrooms. Only with the Full details add-on |
| `energy_label` | string | Energy label, e.g. A, B, A+ |
| `construction_year` | number | Year built |
| `listed_at` | string | When the listing was published, ISO 8601 (the list is sorted on it, newest first; Immoweb and Pararius search results have none, see the README) |
| `agent_name` | string | Estate agent / landlord |
| `agent_phone` | string | Agent phone number when the site shows it |
| `agent_url` | string | Agent page on the site |
| `image_urls` | array | Photo URLs. All photos with the Full details add-on |
| `description` | string | Full description. Only with the Full details add-on |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `original_price` | number | Original asking price when the price was changed (Full details add-on) |
| `site_price_history` | array | The site's own price history of the listing, as published (Full details add-on, when present) |
| `price_condition` | string | kosten_koper (costs on top) or per_month |
| `neighbourhood` | string | Neighbourhood (buurt) |
| `municipality` | string | Municipality (gemeente) |
| `construction_type` | string | resale or newly_built |
| `promo_text` | string | The agent's one-line teaser |
| `details_fetched` | boolean | True when the Full details add-on enriched this item |
| `first_seen_at` | string | Price history add-on: when your history store first saw this listing |
| `previous_price` | number | Price history add-on: the price at the previous run that saw it |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: change in percent |
| `status_changed` | boolean | Price history add-on: status differs from the previous run |
| `price_history` | array | Price history add-on: last 20 price/status points ({at, price, status}) |
| `property_subtype` | string | The site's own subtype, e.g. penthouse, duplex, villa |
| `updated_at` | string | When the listing was last changed on the site, ISO 8601 |
| `price_drop_date` | string | When the last price change happened, ISO 8601 (null if the price never changed) |
| `old_price` | number | Previous price when the listing was marked as price-reduced or changed (null otherwise) |
| `monthly_costs` | number | Rentals: monthly charges on top of the rent, when given |
| `price_max` | number | Upper end of a new-build project's price range (null for single listings) |
| `price_updated` | boolean | True when Immovlan shows the 'Price updated' ribbon on the listing |
| `labels` | array | Ribbons shown on the search card: New, Best of, Price updated, Updated, Option, Sold, Rented |
| `snippet` | string | Shortened description from the search card (available without the Full details add-on) |
| `status_label` | string | Huislijn's status text: Beschikbaar, Onder bod, Verkocht onder voorbehoud |
| `external_url` | string | For rentals fed by another portal: the link to the original listing (e.g. huurwoningen.nl) |
| `furnishing` | string | Furnished, upholstered or shell, as the card says (rentals) |
| `also_on` | array | Other sites that carry the same home (same postcode, house number and listing type); empty when only one does |

## Price

Pay per result: $1.00 per 1,000 listings on the Apify Free plan, less on paid plans, plus $0.025 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- Full details: $2.00 per 1,000. Opens each listing's detail page on Huislijn and Pararius: description, coordinates, area, agent and more. Pararius details are best effort.
- Price history: $1.00 per 1,000. One listing that already had a history in your named store, annotated with previous price, price change and the price/status history. The first run of a listing is free.

Store page, reviews and full documentation: [Netherlands Real Estate Scraper](https://apify.com/carabain-digital/netherlands-real-estate)

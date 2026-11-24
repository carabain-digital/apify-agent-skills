---
name: discogs-marketplace
description: "Scrape the Discogs marketplace: every record copy for sale per release, master or seller. Price, shipping, converted price, media and sleeve condition, seller notes and rating, plus release have/want, last sold and low/median/high price. Reliable, no login. Runs the Apify Actor carabain-digital/discogs-marketplace. Use for tasks like: Cheapest copies of a Discogs release for sale; All versions of a Discogs master for sale."
---

# Discogs Marketplace Scraper

Scrape the Discogs marketplace: every record copy for sale per release, master or seller. Price, shipping, converted price, media and sleeve condition, seller notes and rating, plus release have/want, last sold and low/median/high price. Reliable, no login.

Runs on Apify as the Actor [`carabain-digital/discogs-marketplace`](https://apify.com/carabain-digital/discogs-marketplace?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 3,333 marketplace listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~discogs-marketplace/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~discogs-marketplace/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"start_urls": [{"url": "https://www.discogs.com/release/249504-Rick-Astley-Never-Gonna-Give-You-Up"}], "sort": "price_asc", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/discogs-marketplace").call(run_input={"start_urls": [{"url": "https://www.discogs.com/release/249504-Rick-Astley-Never-Gonna-Give-You-Up"}], "sort": "price_asc", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/discogs-marketplace`.

## Input

| Field | Type | Description |
|---|---|---|
| `start_urls` | array | Release, master or seller pages copied from discogs.com. Example: `https://www.discogs.com/release/249504-Rick-Astley-Never-Gonna-Give-You-Up`. A master (`/master/13509`) is expanded into its versions, a seller page (`/seller/Ronan266/profile`) gives that seller's inventory, `/sell/list?release_id=249504&condition=...&ships_from=...` keeps the filters in the URL. Leave empty to use Search. Format: `[{"url": "https://..."}]`. |
| `search` | string | Free-text search in the Discogs database: the best matching releases are scraped. Example: `daft punk discovery vinyl`. Works together with the URLs above; set Max releases to control how many releases are used. |
| `limit` | integer | Stop after this many marketplace listings in total. Example: `200`. Discogs shows up to 250 copies per request, so large releases are fast. Default: `100`. |
| `sort` | string | Order of the listings per release, by the price converted to the viewer's currency. Example: `price_asc` for the cheapest copies first. A `sort` in your URL wins. Values: `price_asc`, `price_desc`. Default: `"price_asc"`. |
| `max_releases` | integer | For a master (its versions), a search, or a seller page: how many releases to scrape listings for or to add release data (have/want, last sold, price statistics) for. Example: `10`. Default: `10`. |
| `discogs_token` | string | A personal access token from discogs.com/settings/developers raises the release-data rate limit from 25 to 60 requests per minute. Only useful for searches and masters with many releases. Leave empty for normal runs. |

## Examples

**Cheapest copies of a Discogs release for sale**: Scrape every copy for sale of a Discogs release (example: Rick Astley, Never Gonna Give You Up, UK 7 inch): price and shipping, converted price in USD, media and sleeve condition, seller rating and ships-from, plus have/want, last sold and median price. Export to JSON, CSV or Excel.

```json
{
  "start_urls": [
    {
      "url": "https://www.discogs.com/release/249504-Rick-Astley-Never-Gonna-Give-You-Up"
    }
  ],
  "sort": "price_asc",
  "limit": 100
}
```

**All versions of a Discogs master for sale**: Scrape the marketplace listings of every version of a Discogs master release (example: Queen, Back Chat) in one run: price, shipping, media and sleeve condition, seller notes and rating, ships-from, plus per-release have/want, last sold and low/median/high price. Compare pressings and spot underpriced copies.

```json
{
  "start_urls": [
    {
      "url": "https://www.discogs.com/master/13509-Queen-Back-Chat"
    }
  ],
  "max_releases": 10,
  "sort": "price_asc",
  "limit": 200
}
```

## Output

One dataset item per result. Main fields: `title`, `price`, `currency`, `converted_price`, `shipping_price`, `media_condition`, `sleeve_condition`, `seller_name`, `seller_rating_pct`, `ships_from`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `discogs:<listing id>` |
| `source` | string | Always `discogs` |
| `url` | string | Listing page on discogs.com (`/sell/item/<id>`) |
| `listing_id` | string | Discogs marketplace listing ID |
| `release_id` | string | Discogs release ID this copy belongs to |
| `release_url` | string | Release page on discogs.com |
| `title` | string | Listing title as shown on the marketplace, `Artist - Title (format)` |
| `label` | string | Record label of the pressing |
| `catalog_number` | string | Catalog number of the pressing |
| `price` | number | Asking price in the seller's currency (`currency`), without shipping |
| `currency` | string | Currency code of `price` and `shipping_price` (e.g. GBP, EUR, USD, SEK) |
| `shipping_price` | number | Shipping cost to the buyer country of the proxy (US by default) in `currency`; 0 = free, null = the seller does not ship there |
| `converted_price` | number | Price converted by Discogs to the viewer's currency (USD with the default US proxy), the figure to sort and compare on |
| `converted_currency` | string | Currency of `converted_price` |
| `media_condition` | string | Condition of the record/disc, e.g. `Near Mint (NM or M-)`, `Very Good Plus (VG+)` |
| `sleeve_condition` | string | Condition of the sleeve, or `Generic`, `No Cover`, `Not Graded` |
| `seller_comments` | string | The seller's own notes on this copy |
| `seller_name` | string | Seller's Discogs username (public store name) |
| `seller_url` | string | Seller's public profile on discogs.com |
| `seller_rating_pct` | number | Seller's positive feedback in percent |
| `seller_rating_count` | integer | Number of ratings the seller has |
| `ships_from` | string | Country the seller ships from |
| `unavailable_in` | string | Country in which Discogs says the copy cannot be bought (the buyer country of the proxy), else null |
| `accepts_offers` | boolean | True when the seller accepts offers (`Make Offer`) |
| `scraped_at` | string | UTC timestamp of the scrape |
| `artist` | string | Release artist (Discogs API) |
| `release_title` | string | Release title (Discogs API) |
| `year` | integer | Release year |
| `country` | string | Release country of pressing |
| `formats` | array | Release formats, e.g. `Vinyl (7", 45 RPM, Single, Stereo)` |
| `genres` | array | Release genres |
| `styles` | array | Release styles |
| `master_id` | integer | Master release ID, when the release belongs to one |
| `release_have` | integer | Discogs users who have the release in their collection |
| `release_want` | integer | Discogs users who want the release |
| `release_rating` | number | Average community rating of the release (0-5) |
| `release_rating_count` | integer | Number of community ratings |
| `release_num_for_sale` | integer | Copies for sale of the release right now (all sellers) |
| `release_lowest_price` | number | Lowest asking price right now, in USD (Discogs API) |
| `release_last_sold` | string | Date the release last sold (YYYY-MM-DD), null when it never sold |
| `release_price_low` | number | Lowest price paid in the past, from the release page statistics |
| `release_price_median` | number | Median price paid |
| `release_price_high` | number | Highest price paid |
| `release_price_currency` | string | Currency of the three statistics prices (USD with the default US proxy) |

## Price

Pay per result: $1.50 per 1,000 marketplace listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.

Store page, reviews and full documentation: [Discogs Marketplace Scraper](https://apify.com/carabain-digital/discogs-marketplace?fpr=4ijnom)

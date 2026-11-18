---
name: france-real-estate
description: "Search SeLoger, Bien'ici and PAP at once for annonces immobilières in France: appartements and maisons, location or vente. One search, one dataset, merged and newest first. Optional full details and price history. Residential proxy included. JSON, CSV or Excel. Runs the Apify Actor carabain-digital/france-real-estate. Use for tasks like: Appartements à vendre à Lyon, three portals; Locations in Paris with price history."
---

# France Real Estate Scraper

Search SeLoger, Bien'ici and PAP at once for annonces immobilières in France: appartements and maisons, location or vente. One search, one dataset, merged and newest first. Optional full details and price history. Residential proxy included. JSON, CSV or Excel.

Runs on Apify as the Actor [`carabain-digital/france-real-estate`](https://apify.com/carabain-digital/france-real-estate), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 3,333 listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~france-real-estate/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~france-real-estate/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"location": "Lyon", "listing_type": "sale", "property_type": "apartment", "limit": 100}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/france-real-estate").call(run_input={"location": "Lyon", "listing_type": "sale", "property_type": "apartment", "limit": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/france-real-estate`.

## Input

| Field | Type | Description |
|---|---|---|
| `location` | string | French city, arrondissement, department or postcode, sent to all sites. Several places separated by commas run as separate searches. Example: `Lyon`, `Paris 15e, Nice` or `69003`. PAP only searches inside a place: without a location PAP is skipped. Leave empty for all of France on SeLoger and Bien'ici. |
| `listing_type` | string | Search homes `rent` or `sale`. Example: `rent`. Values: `sale`, `rent`. Default: `"sale"`. |
| `min_price` | integer | Lowest asking price in euro (rent per month; SeLoger and PAP show rent charges included, Bien'ici excluded). Example: `800` (rent) or `200000` (sale). |
| `max_price` | integer | Highest asking price in euro (rent per month). Example: `1500` (rent) or `450000` (sale). |
| `property_type` | string | Only this type of property. Example: `apartment`. `any` returns every type the sites list (SeLoger: houses and apartments). Values: `any`, `house`, `apartment`, `land`, `commercial`, `parking`. Default: `"any"`. |
| `min_rooms` | integer | Only homes with at least this many rooms (pièces, a studio counts 1). Example: `3`. |
| `min_living_area_m2` | integer | Only homes with at least this living area in square metres. Example: `60`. |
| `sites` | array | Which sites to search. Default: all of them. Example: `["seloger", "bienici", "pap"]`. Default: `["seloger", "bienici", "pap"]`. |
| `limit` | integer | Cap on the merged dataset, newest first. Example: `100`. Each site has its own ceiling: SeLoger 10,000, Bien'ici 2,500 and PAP about 300-375 results per search. Default: `100`. |
| `limit_per_site` | integer | Optional cap per site, to keep the mix balanced and browser sites short. Default: Max listings. Example: `25`. |
| `includeDetails` | boolean | Also open each listing's page on the selected sites: full description, coordinates (when the exact address is public), rent without charges, deposit, DPE values, agency contact and all photos. Adds $2.00 per 1,000 listings that were actually enriched. Runs take a little longer. Default: `false`. |
| `trackHistory` | boolean | Keep a price and status history per listing in a named storage in your own account and add `first_seen_at`, `previous_price`, `price_change`, `price_change_pct`, `status_changed` and `price_history` to every listing you have scraped before. The history store is shared with our single-site real-estate Actors. Adds $1.00 per 1,000 listings that already had history (the first run only starts the series). Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (a-z, 0-9 and dashes). Use the same name in all our real-estate Actors to share one history. Example: `realestate-history`. Default: `"realestate-history"`. |

## Examples

**Appartements à vendre à Lyon, three portals**: The newest apartments for sale in Lyon from SeLoger, Bien'ici and PAP in one list, with price, address, area and agency.

```json
{
  "location": "Lyon",
  "listing_type": "sale",
  "property_type": "apartment",
  "limit": 100
}
```

**Locations in Paris with price history**: The newest rentals in Paris from SeLoger, Bien'ici and PAP, with price changes since your last run. Schedule it daily for a rent-drop feed.

```json
{
  "location": "Paris",
  "listing_type": "rent",
  "limit": 300,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `listed_at`, `source`, `title`, `city`, `price`, `property_type`, `living_area_m2`, `rooms`, `also_on`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `<source>:<site id>` |
| `source` | string | Site the item was taken from: seloger, bienici, pap |
| `url` | string | Public listing URL. Belles Demeures listings that appear in SeLoger results keep their own bellesdemeures.com URL |
| `title` | string | Listing headline as the agent wrote it |
| `listing_type` | string | sale or rent |
| `property_type` | string | house, apartment, land, commercial, parking or other (for example new-build programmes) |
| `property_subtype` | string | The site's own subtype in lower case, e.g. studio, bungalow, town-house, office-space, store, restaurant |
| `status` | string | available, under_offer, sold or rented |
| `price` | number | Asking price in EUR (rent per month for rentals). Null when on request. SeLoger rents mostly include charges, see price_includes_charges |
| `price_currency` | string | ISO currency code (EUR) |
| `price_period` | string | month for rentals, null for sales |
| `price_type` | string | asking or on_request |
| `price_includes_charges` | boolean | Rentals: true when the price is the rent charges included (charges comprises), false when excluding charges (hors charges). Null for sales |
| `address` | string | Postcode and city on one line. SeLoger never publishes the street |
| `street` | string | Always null: SeLoger does not publish street names |
| `house_number` | string | House number. None of the three portals publish it, so this is nearly always null |
| `postcode` | string | Postcode (code postal), e.g. 69003 |
| `city` | string | City / municipality without the arrondissement, e.g. Paris, Lyon |
| `district` | string | Quarter (quartier) as the site reports it, e.g. Jardin des Plantes; the arrondissement when there is no quarter |
| `region` | string | French region as the site reports it, e.g. Ile-de-France, Auvergne-Rhône-Alpes |
| `department` | string | French department as the site reports it, e.g. Paris, Rhône |
| `country` | string | ISO country code (FR) |
| `lat` | number | Latitude. Add-on Full details only, and only when the exact address is public |
| `lon` | number | Longitude. Add-on Full details only, and only when the exact address is public |
| `living_area_m2` | number | Living area in m2 |
| `plot_area_m2` | number | Plot area in m2 |
| `rooms` | number | Number of rooms (pièces), living room included |
| `bedrooms` | integer | Number of bedrooms, when the agent gave it |
| `bathrooms` | integer | Not published as a number by SeLoger: always null |
| `energy_label` | string | DPE energy class (A to G). Null when the listing has no energy diagnosis yet |
| `ghg_label` | string | Add-on Full details: GES greenhouse gas class (A to G) from the energy diagnosis |
| `construction_year` | integer | Year of construction, when the agent gave it |
| `condition` | string | Condition as the site reports it, e.g. mint-condition, well-kept, refurbished, need-of-renovation. Add-on Full details can fill it in French (e.g. Comme neuf) when the search data has none |
| `heating` | string | Add-on Full details: heating, e.g. Chauffage individuel |
| `energy_source` | string | Add-on Full details: main energy source, e.g. Gaz, Électricité |
| `energy_consumption_kwh_m2` | number | Add-on Full details: primary energy consumption from the DPE in kWh per m2 and year |
| `ghg_emissions_kg_m2` | number | Add-on Full details: greenhouse gas emissions from the diagnosis in kg CO2 per m2 and year |
| `available_from` | string | Move-in date as ISO date, or `immediately`, when the agent gave it |
| `deposit` | number | Add-on Full details, rentals: deposit (dépôt de garantie) in EUR, when given |
| `features` | array | Add-on Full details: amenities as listed on the page (cellar, garden, parking, alarm, ...) |
| `listed_at` | string | When the listing was published, ISO 8601 (the list is sorted on it, newest first). PAP has a date only with the Full details add-on (date only, YYYY-MM-DD) |
| `updated_at` | string | When the listing was last changed on the site, ISO 8601 |
| `agent_name` | string | Estate agency or developer. Null for private owners |
| `agent_phone` | string | Agent phone number as shown on the listing |
| `agent_url` | string | Agency page on seloger.com |
| `image_urls` | array | Photo URLs (all photos) |
| `description` | string | Listing description. The search data carries a short text; the add-on Full details gives the complete text |
| `monthly_costs` | number | Add-on Full details, rentals: monthly charges (charges / provision pour charges) as the agent split them out, when given |
| `rent_excl_charges` | number | Add-on Full details, rentals: rent excluding charges (loyer hors charges), when given |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |
| `first_seen_at` | string | Price history add-on: when this listing was first seen by your runs |
| `previous_price` | number | Price history add-on: the price at the previous run (null on the first run) |
| `price_change` | number | Price history add-on: price minus previous price |
| `price_change_pct` | number | Price history add-on: price change in percent |
| `status_changed` | boolean | Price history add-on: true when the status changed since the previous run |
| `price_history` | array | Price history add-on: price and status points over your runs, `[{at, price, status}]`, last 20 |
| `old_price` | number | Always null on Bien'ici (it only flags price decreases, without the old price) |
| `also_on` | array | Other sites that carry the same home (same postcode, house number and listing type); empty when only one does. Rare in France: no portal publishes house numbers |
| `details_fetched` | boolean | True when the Full details add-on enriched this item |

## Price

Pay per result: $1.50 per 1,000 listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $2.00 per 1,000. Opens each listing's page on SeLoger, Bien'ici and PAP: full description, coordinates, rent without charges, DPE values, agency contact, all photos and more.
- Price history: $1.00 per 1,000. One listing that already had a history in your named store, annotated with previous price, price change and the price/status history. The first run of a listing is free.

Store page, reviews and full documentation: [France Real Estate Scraper](https://apify.com/carabain-digital/france-real-estate)

---
name: seloger-listings
description: "Scrape real-estate listings from SeLoger (seloger.com), France's biggest property portal: appartements, maisons, terrains and commercial property for sale and for rent. Filter by city, department or postcode, price, rooms and area. Newest first, with agent phone. Add-on: full details and DPE data. Runs the Apify Actor carabain-digital/seloger-listings. Use for tasks like: Appartements à louer à Paris: newest rentals on SeLoger; Maisons à vendre à Lyon with price history on SeLoger."
---

# SeLoger Scraper - French Property Listings

Scrape real-estate listings from SeLoger (seloger.com), France's biggest property portal: appartements, maisons, terrains and commercial property for sale and for rent. Filter by city, department or postcode, price, rooms and area. Newest first, with agent phone. Add-on: full details and DPE data.

Runs on Apify as the Actor [`carabain-digital/seloger-listings`](https://apify.com/carabain-digital/seloger-listings?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 5,882 listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~seloger-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~seloger-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"location": "Paris", "listing_type": "rent", "property_type": "apartment", "max_price": 2000, "limit": 100, "includeDetails": false}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/seloger-listings").call(run_input={"location": "Paris", "listing_type": "rent", "property_type": "apartment", "max_price": 2000, "limit": 100, "includeDetails": false})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/seloger-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `location` | string | French city, department, region or postcode. Several values separated by commas. Example: `Lyon`, `Paris, Nice`, `Rhône` or `75011`. Add the department to pick a homonym: `Saint-Denis (93)`. Required unless you give search URLs. Ignored when you give search URLs. |
| `listing_type` | string | `sale` for properties to buy, `rent` for rentals. Example: `rent`. Values: `sale`, `rent`. Default: `"sale"`. |
| `property_type` | string | Type of property. `any` = houses and apartments (maisons et appartements). Example: `apartment`. Values: `any`, `house`, `apartment`, `land`, `commercial`, `parking`. Default: `"any"`. |
| `min_price` | integer | Lowest price in euro (rent: monthly rent as shown on the site, usually charges included). Example: `200000`, for rent `800`. |
| `max_price` | integer | Highest price in euro (rent: monthly rent as shown on the site, usually charges included). Example: `450000`, for rent `1500`. |
| `min_rooms` | number | Minimum number of rooms (pièces, living room included). Example: `3`. |
| `min_living_area_m2` | integer | Minimum living area in square metres. Example: `60`. |
| `start_urls` | array | Search result pages copied from seloger.com; they replace the filters above. Example: `https://www.seloger.com/classified-search?distributionTypes=Rent&estateTypes=Apartment&locations=AD08FR31096&priceMax=1500`. Filters in the URL are kept; an `/immobilier/achat/immo-lyon-69/bien-appartement/` style page works too. Leave empty to use the filters above. Format: `[{"url": "https://..."}]`. |
| `limit` | integer | Stop after this many listings (newest first). Example: `100`. SeLoger shows at most 10,000 results per search: use a city, arrondissement or price range to go deeper. Default: `100`. |
| `includeDetails` | boolean | Also open each listing's page: full description, coordinates (when the exact address is public), rent without charges, monthly charges, deposit, DPE and GES values, heating, amenities and all photos. Adds $1.00 per 1,000 listings on top of the base price. One extra request per listing, so runs take a little longer. Default: `false`. |
| `trackHistory` | boolean | Keep the price and status of every listing in a named store in your own Apify account and add first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history to each item. Run it on a schedule: the first run only starts the series. Adds $1.00 per 1,000 listings that already had history. Do not run two runs on the same store at once. Default: `false`. |
| `historyStoreName` | string | Name of the key-value store that holds the history (letters a-z, digits, dashes). All our real-estate Actors use `realestate-history` by default, so history is shared between them. Use another name to keep a separate series per search, e.g. `gent-apartments`. Default: `"realestate-history"`. |

## Examples

**Appartements à louer à Paris: newest rentals on SeLoger**: Scrape the newest apartments for rent in Paris (appartements à louer) from SeLoger (seloger.com): monthly rent, charges included or not, arrondissement, quarter, rooms, living area, DPE energy class and agency phone. Export to JSON, CSV or Excel.

```json
{
  "location": "Paris",
  "listing_type": "rent",
  "property_type": "apartment",
  "max_price": 2000,
  "limit": 100,
  "includeDetails": false
}
```

**Maisons à vendre à Lyon with price history on SeLoger**: Scrape houses for sale in Lyon (maisons à vendre) from SeLoger (seloger.com) and track their price over time: asking price, price changes, quarter, living and plot area, DPE class and agency. Run it daily to catch price drops.

```json
{
  "location": "Lyon",
  "listing_type": "sale",
  "property_type": "house",
  "limit": 200,
  "includeDetails": false,
  "trackHistory": true
}
```

## Output

One dataset item per result. Main fields: `updated_at`, `title`, `city`, `price`, `living_area_m2`, `rooms`, `status`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `<source>:<site id>` |
| `source` | string | Short site name (seloger) |
| `url` | string | Public listing URL. Belles Demeures listings that appear in SeLoger results keep their own bellesdemeures.com URL |
| `title` | string | Listing headline as the agent wrote it |
| `listing_type` | string | sale or rent |
| `property_type` | string | house, apartment, land, commercial, parking or other (for example new-build programmes) |
| `property_subtype` | string | The site's own subtype in lower case, e.g. studio, bungalow, town-house, office-space, store, restaurant |
| `status` | string | available, under_offer, sold or rented |
| `price` | number | Asking price in EUR (sale) or monthly rent (rent) as shown on the site. For most rentals this is the rent charges included, see price_includes_charges. Null when the price is on request (many new-build programmes) |
| `price_currency` | string | ISO currency code (EUR) |
| `price_period` | string | month for rentals, null for sales |
| `price_type` | string | asking or on_request |
| `price_includes_charges` | boolean | Rentals: true when the price is the rent charges included (charges comprises), false when excluding charges (hors charges). Null for sales |
| `address` | string | Postcode and city on one line. SeLoger never publishes the street |
| `street` | string | Always null: SeLoger does not publish street names |
| `house_number` | string | Always null: SeLoger does not publish house numbers |
| `postcode` | string | French postcode (5 digits). For Paris, Lyon and Marseille it identifies the arrondissement |
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
| `listed_at` | string | When the listing was first published on SeLoger, ISO 8601 |
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

## Price

Pay per result: $0.85 per 1,000 listings on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One listing enriched with its detail page: full description, coordinates (when the exact address is public), rent without charges, monthly charges, deposit, construction year, heating, DPE and GES values, amenities and all photos.
- Price history: $1.00 per 1,000. One listing that already had history in your named history store, enriched with first_seen_at, previous_price, price_change, price_change_pct, status_changed and price_history. The first run of a listing is free.

Store page, reviews and full documentation: [SeLoger Scraper - French Property Listings](https://apify.com/carabain-digital/seloger-listings?fpr=4ijnom)

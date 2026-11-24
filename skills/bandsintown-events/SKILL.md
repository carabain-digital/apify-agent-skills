---
name: bandsintown-events
description: "Scrape concert and tour dates from Bandsintown (bandsintown.com): paste artist pages or type artist names and get every upcoming event with date, time, venue, city, country, coordinates, ticket link and status. Add-on: street address, full lineup and ticket sellers. Runs the Apify Actor carabain-digital/bandsintown-events. Use for tasks like: Usher tour dates and tickets from Bandsintown; Tour calendar for several artists on Bandsintown."
---

# Bandsintown Events Scraper

Scrape concert and tour dates from Bandsintown (bandsintown.com): paste artist pages or type artist names and get every upcoming event with date, time, venue, city, country, coordinates, ticket link and status. Add-on: street address, full lineup and ticket sellers.

Runs on Apify as the Actor [`carabain-digital/bandsintown-events`](https://apify.com/carabain-digital/bandsintown-events?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 1,666 events with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~bandsintown-events/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~bandsintown-events/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"artists": ["Usher"], "limit": 100, "includeDetails": true}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/bandsintown-events").call(run_input={"artists": ["Usher"], "limit": 100, "includeDetails": true})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/bandsintown-events`.

## Input

| Field | Type | Description |
|---|---|---|
| `startUrls` | array | Artist pages or event pages copied from bandsintown.com. Example: `https://www.bandsintown.com/a/497-usher` (all upcoming events of Usher) or `https://www.bandsintown.com/e/108775697-usher-at-mercedes-benz-stadium` (one event). The number after `/a/` decides the artist, the name part is ignored. Format: `[{"url": "https://..."}]`. |
| `artists` | array | Look artists up by name (the best match on Bandsintown is used, an exact name wins) or give the numeric artist ID. Example: `Pitbull`, `Coldplay` or `497`. Used in addition to the URLs above. |
| `includePast` | boolean | Also return the artist's most recent past events (up to 50 per artist: the site shows no more without a login). They have no coordinates and no status. Example: `false`. Default: `false`. |
| `limit` | integer | Stop after this many events in total, over all artists. Example: `100`. Default: `100`. |
| `includeDetails` | boolean | Also open each event's page for the street address and postcode, venue link, the full lineup, free-admission flag and the list of ticket sellers. The date, venue, city, country, coordinates, ticket link and status are already in the base data. Adds $1.00 per 1,000 events on top of the base price. One extra request per event, so runs take longer. Default: `false`. |

## Examples

**Usher tour dates and tickets from Bandsintown**: Scrape all upcoming Usher concerts from Bandsintown (bandsintown.com): date and time, venue, street address, city, country, coordinates, lineup, sold-out flag and ticket link. Export to JSON, CSV or Excel.

```json
{
  "artists": [
    "Usher"
  ],
  "limit": 100,
  "includeDetails": true
}
```

**Tour calendar for several artists on Bandsintown**: Build a concert calendar for a list of artists from Bandsintown (bandsintown.com): every upcoming date with venue, city, country, coordinates, status and ticket link. Run it weekly to catch new tour dates and cancellations.

```json
{
  "artists": [
    "Pitbull",
    "Coldplay",
    "Metallica",
    "Taylor Swift"
  ],
  "limit": 500,
  "includeDetails": false
}
```

## Output

One dataset item per result. Main fields: `starts_at`, `artist`, `venue`, `city`, `country`, `status`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable event ID, `bandsintown:<event id>` |
| `source` | string | Always `bandsintown` |
| `url` | string | Public event page on bandsintown.com |
| `name` | string | Event name: the festival or tour title when there is one, otherwise `<artist> @ <venue>` |
| `artist` | string | Artist whose page the event was found on |
| `artist_id` | string | Bandsintown artist ID |
| `artist_url` | string | Artist page, `https://www.bandsintown.com/a/<id>` |
| `starts_at` | string | Start in the venue's local time, ISO 8601 without offset, for example `2026-11-05T19:00:00` |
| `date` | string | Local start date, YYYY-MM-DD |
| `time` | string | Local start time, HH:MM |
| `ends_at` | string | Local end time when the event has one, otherwise null |
| `timezone` | string | IANA time zone of the venue, for example `America/New_York` |
| `is_past` | boolean | True for events that already happened (only with Include past events) |
| `venue` | string | Venue name |
| `venue_url` | string | Venue page on Bandsintown. Only with the Full details add-on |
| `street` | string | Venue street address. Only with the Full details add-on |
| `postcode` | string | Venue postcode. Only with the Full details add-on |
| `city` | string | City |
| `region` | string | US state or Canadian province code (`GA`, `ON`) for events there; with Full details also other regions when the site has one |
| `country` | string | Country name (`United States`, `Germany`) |
| `lat` | number | Venue latitude. Upcoming events only; past events have none unless Full details is on |
| `lon` | number | Venue longitude, see lat |
| `lineup` | array | Artist names on the bill. Without Full details only the artist whose page it is; with it the whole lineup |
| `status` | string | scheduled, canceled, postponed or rescheduled. Null for past events |
| `sold_out` | boolean | True when the site shows the event as sold out. Null for past events |
| `free` | boolean | True for free admission. Only with the Full details add-on |
| `ticket_url` | string | Bandsintown ticket link `https://www.bandsintown.com/t/<event id>`, which forwards to a seller. Null when no tickets are offered |
| `ticket_sellers` | array | Ticket sellers listed on the event page: `[{seller, type, price, currency, url}]`. Only with the Full details add-on. Prices are mostly null: Bandsintown rarely shows them |
| `scraped_at` | string | When this item was scraped, UTC ISO 8601 |

## Price

Pay per result: $3.00 per 1,000 events on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Full details: $1.00 per 1,000. One event enriched from its event page: street address, postcode, venue link, full lineup, free-admission flag and ticket sellers.

Store page, reviews and full documentation: [Bandsintown Events Scraper](https://apify.com/carabain-digital/bandsintown-events?fpr=4ijnom)

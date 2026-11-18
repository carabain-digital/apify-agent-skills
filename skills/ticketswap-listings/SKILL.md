---
name: ticketswap-listings
description: "Scrape resale ticket listings from TicketSwap: price per ticket with and without fees, face value, quantity, seats, seller note, ticket type, event, date, venue and city. Event, ticket-type or city URLs; available, reserved and sold listings; any currency. Runs the Apify Actor carabain-digital/ticketswap-listings. Use for tasks like: Amsterdam resale ticket prices on TicketSwap; Berlin sold and available tickets on TicketSwap."
---

# TicketSwap Listings Scraper

Scrape resale ticket listings from TicketSwap: price per ticket with and without fees, face value, quantity, seats, seller note, ticket type, event, date, venue and city. Event, ticket-type or city URLs; available, reserved and sold listings; any currency.

Runs on Apify as the Actor [`carabain-digital/ticketswap-listings`](https://apify.com/carabain-digital/ticketswap-listings), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,500 listings with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~ticketswap-listings/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~ticketswap-listings/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"startUrls": ["https://www.ticketswap.com/city/amsterdam/3"], "maxEvents": 10, "limit": 50, "statuses": ["available"]}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/ticketswap-listings").call(run_input={"startUrls": ["https://www.ticketswap.com/city/amsterdam/3"], "maxEvents": 10, "limit": 50, "statuses": ["available"]})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/ticketswap-listings`.

## Input

| Field | Type | Description |
|---|---|---|
| `startUrls` | array | TicketSwap URLs: an event page (all ticket types), a ticket-type page (only that type), or a city / category page to pick up the events it shows first. Example: `https://www.ticketswap.com/concert-tickets/muse-amsterdam-ziggo-dome-2026-11-30-Cby6zngFKckmfBWkYEuk6` or `https://www.ticketswap.com/city/amsterdam/3`. |
| `statuses` | array | Which listings to collect. Example: `["available", "sold"]`. TicketSwap shows all available listings but only the 3 most recent reserved and sold listings per ticket type. Default: `["available"]`. |
| `limit` | integer | Stop collecting an event after this many listings. Ticket types are opened in the site's order; within a type the listings come cheapest first. Example: `50`. Default: `50`. |
| `maxEvents` | integer | For city or category start URLs: how many of the events shown on that page to open. Event and ticket-type URLs are always opened. Example: `10`. Default: `10`. |
| `currency` | string | Show prices in this currency, as a 3-letter code TicketSwap offers in its currency menu. Example: `GBP`, `USD`, `DKK`. Empty = the site's default (EUR). The seller's own currency and price are always included too. |

## Examples

**Amsterdam resale ticket prices on TicketSwap**: Scrape the resale listings of the first 10 events on TicketSwap's Amsterdam page: price per ticket with and without fees, face value, number of tickets, seats, ticket type, venue and date. Schedule it to track resale prices, and export to JSON, CSV or Excel.

```json
{
  "startUrls": [
    "https://www.ticketswap.com/city/amsterdam/3"
  ],
  "maxEvents": 10,
  "limit": 50,
  "statuses": [
    "available"
  ]
}
```

**Berlin sold and available tickets on TicketSwap**: Scrape available and recently sold resale tickets for the first 10 events on TicketSwap's Berlin page: what buyers pay per ticket, face value, quantity, ticket type and venue. Compare asking prices with real sale prices, and export to JSON, CSV or Excel.

```json
{
  "startUrls": [
    "https://www.ticketswap.com/city/berlin/2"
  ],
  "maxEvents": 10,
  "limit": 50,
  "statuses": [
    "available",
    "sold"
  ]
}
```

## Output

One dataset item per result. Main fields: `event_name`, `event_date`, `city`, `ticket_type`, `status`, `price_per_ticket`, `price_currency`, `tickets_available`, `url`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable listing ID, `ticketswap:<listing number>` |
| `source` | string | Short site name (`ticketswap`) |
| `url` | string | Public listing URL |
| `status` | string | available, reserved or sold |
| `event_name` | string | Event name |
| `event_url` | string | Event page URL |
| `event_date` | string | Start of the ticket type's event day (ISO 8601 with offset) |
| `event_category` | string | TicketSwap category: concerts, festivals, clubs, sports ... |
| `venue` | string | Venue name |
| `city` | string | Venue city |
| `country` | string | Venue country code (ISO 3166-1 alpha-2) |
| `ticket_type` | string | Ticket type, e.g. Regular, Seats, Friday Entry |
| `ticket_type_group` | string | Group the ticket type sits in on the event page (e.g. a festival day), or null |
| `ticket_type_url` | string | Ticket-type page URL |
| `price_per_ticket` | number | Price per ticket the buyer pays, fees included, in `price_currency` |
| `seller_price_per_ticket` | number | Price per ticket the seller asks (without TicketSwap fees), in `price_currency` |
| `price_currency` | string | Currency of `price_per_ticket` and `seller_price_per_ticket` (the selected currency, default EUR) |
| `face_value_per_ticket` | number | Original (face) value per ticket as entered by the seller |
| `face_value_currency` | string | Currency of the face value |
| `seller_currency_price_per_ticket` | number | Buyer price per ticket in the seller's own currency |
| `seller_currency` | string | The seller's currency |
| `tickets_available` | integer | Tickets still for sale in this listing |
| `tickets_in_listing` | integer | Tickets in this listing |
| `description` | string | Seller's note, or null |
| `seating` | array | Seat per ticket for seated events: entrance, section, row, seat. Empty for standing tickets |
| `type_available_tickets` | integer | Tickets available for this ticket type at scrape time |
| `type_sold_tickets` | integer | Tickets sold on TicketSwap for this ticket type |
| `scraped_at` | string | When the listing was scraped (UTC ISO 8601) |

## Price

Pay per result: $2.00 per 1,000 listings on the Apify Free plan, less on paid plans, plus $0.025 per run start per GB of memory. No subscription; Apify bills the run to the user's account.

Store page, reviews and full documentation: [TicketSwap Listings Scraper](https://apify.com/carabain-digital/ticketswap-listings)

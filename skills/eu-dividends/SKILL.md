---
name: eu-dividends
description: "Dividend history and splits per ISIN for European and UK shares: ex-date, pay date, declaration date, type, status and the declared amount and currency, merged from Yahoo Finance, Dividendmax and Borsa Italiana. Built for portfolio trackers. Runs the Apify Actor carabain-digital/eu-dividends. Use for tasks like: AEX dividend history and upcoming dividends; Euro Stoxx blue chips: full dividend history."
---

# EU Dividends & Splits by ISIN

Dividend history and splits per ISIN for European and UK shares: ex-date, pay date, declaration date, type, status and the declared amount and currency, merged from Yahoo Finance, Dividendmax and Borsa Italiana. Built for portfolio trackers.

Runs on Apify as the Actor [`carabain-digital/eu-dividends`](https://apify.com/carabain-digital/eu-dividends?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 5,000 dividend or splits with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~eu-dividends/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~eu-dividends/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"isins": ["NL0010273215", "GB00BP6MXD84", "GB00BVZK7T90", "NL0011821202", "NL0000009165", "NL0011794037", "NL0000009538"], "from_date": "2020-01-01", "includeSplits": true, "includeForecasts": true}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/eu-dividends").call(run_input={"isins": ["NL0010273215", "GB00BP6MXD84", "GB00BVZK7T90", "NL0011821202", "NL0000009165", "NL0011794037", "NL0000009538"], "from_date": "2020-01-01", "includeSplits": true, "includeForecasts": true})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/eu-dividends`.

## Input

| Field | Type | Description |
|---|---|---|
| `isins` | array | ISINs of the shares, one per line. Example: `NL0010273215` (ASML), `GB00BP6MXD84` (Shell), `IT0003128367` (Enel). Up to 1,000 per run. The primary listing is looked up on Yahoo Finance; ISINs Yahoo does not know are skipped and logged. |
| `from_date` | string | Only events with an ex-date on or after this day, YYYY-MM-DD. Example: `2024-01-01`. Leave empty for the full history (Yahoo goes back to the 1990s for most large caps). |
| `includeSplits` | boolean | Also return stock splits and reverse splits (consolidations) as events, with the ratio. Default: `true`. |
| `includeForecasts` | boolean | Also return Dividendmax's forecast dividends for the next 12 months: expected ex-date, pay date and type, without an amount (forecast amounts are not public). Default: `false`. |
| `limit` | integer | Stop after this many events in total. Example: `500`. Leave empty for all events of all ISINs. |

## Examples

**AEX dividend history and upcoming dividends**: Dividend history from 2020 and announced dividends of large Amsterdam-listed shares (ASML, Shell, Unilever, ING, Heineken, Ahold Delhaize, Philips): ex-date, pay date, type and the declared amount and currency, plus splits. Export to JSON, CSV or Excel.

```json
{
  "isins": [
    "NL0010273215",
    "GB00BP6MXD84",
    "GB00BVZK7T90",
    "NL0011821202",
    "NL0000009165",
    "NL0011794037",
    "NL0000009538"
  ],
  "from_date": "2020-01-01",
  "includeSplits": true,
  "includeForecasts": true
}
```

**Euro Stoxx blue chips: full dividend history**: Full dividend and split history of European blue chips (TotalEnergies, SAP, Nestlé, Enel, Santander, AB InBev, Novo Nordisk, Equinor) by ISIN: ex-date, pay date, declared amount and currency, and Yahoo's split-adjusted amount for total-return maths. Export to JSON, CSV or Excel.

```json
{
  "isins": [
    "FR0000120271",
    "DE0007164600",
    "CH0038863350",
    "IT0003128367",
    "ES0113900J37",
    "BE0974293251",
    "DK0062498333",
    "NO0010096985"
  ],
  "includeSplits": true,
  "includeForecasts": false
}
```

## Output

One dataset item per result. Main fields: `isin`, `name`, `event`, `status`, `dividend_type`, `ex_date`, `pay_date`, `amount`, `currency`, `split_ratio`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable event ID, `<ISIN>:<event>:<ex-date>` |
| `event` | string | dividend, split or reverse_split |
| `isin` | string | ISIN you asked for |
| `ticker` | string | Yahoo Finance symbol of the primary listing, e.g. ASML.AS |
| `name` | string | Company name |
| `exchange` | string | Exchange of the primary listing, e.g. Amsterdam |
| `listing_currency` | string | Trading currency of that listing, ISO code (London quotes in pence are given in GBP) |
| `status` | string | paid, declared (announced, not paid yet) or forecast (Dividendmax estimate, dates only) |
| `dividend_type` | string | As the source labels it: quarterly, interim, final, special, ordinary, ...; null when only Yahoo has the event |
| `declaration_date` | string | Day the dividend was announced, YYYY-MM-DD |
| `ex_date` | string | Ex-dividend date (for splits: the day the split took effect), YYYY-MM-DD |
| `pay_date` | string | Payment date, YYYY-MM-DD |
| `amount` | number | Dividend per share in `currency`. Declared, unadjusted amount when `amount_is_declared` is true; otherwise Yahoo's split-adjusted amount |
| `currency` | string | ISO currency of `amount` (the declared currency, e.g. USD for Shell) |
| `amount_is_declared` | boolean | true when `amount` is the amount the company declared (Dividendmax / Borsa Italiana), false when it is Yahoo's adjusted amount |
| `adjusted_amount` | number | Yahoo's split-adjusted dividend per share in `adjusted_currency` (the listing currency); null when Yahoo does not have the event |
| `adjusted_currency` | string | Currency of `adjusted_amount` |
| `split_ratio` | string | Split ratio as new:old, e.g. 3:1 (split) or 8:9 (reverse split) |
| `split_from` | number | Old shares in the ratio |
| `split_to` | number | New shares in the ratio |
| `sources` | array | Sources that reported this event: yahoo, dividendmax, borsaitaliana |
| `source_urls` | array | Public pages of those sources |
| `scraped_at` | string | When the run collected this event, UTC ISO 8601 |

## Price

Pay per result: $1.00 per 1,000 dividend or splits on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.

Store page, reviews and full documentation: [EU Dividends & Splits by ISIN](https://apify.com/carabain-digital/eu-dividends?fpr=4ijnom)

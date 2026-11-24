---
name: eu-short-positions
description: "Disclosed net short positions in European and UK shares from nine regulators (AFM, AMF, FCA, BaFin, CONSOB, FI, Finanstilsynet NO/DK, FSMA): holder, issuer, ISIN, net short %, position date. Current positions or the full history. Runs the Apify Actor carabain-digital/eu-short-positions. Use for tasks like: All current net short positions in Europe; Hedge fund short tracker: Citadel, Marshall Wace, AQR."
---

# EU Short Positions Register

Disclosed net short positions in European and UK shares from nine regulators (AFM, AMF, FCA, BaFin, CONSOB, FI, Finanstilsynet NO/DK, FSMA): holder, issuer, ISIN, net short %, position date. Current positions or the full history.

Runs on Apify as the Actor [`carabain-digital/eu-short-positions`](https://apify.com/carabain-digital/eu-short-positions?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,500 short positions with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~eu-short-positions/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~eu-short-positions/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"countries": [], "includeHistory": false}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/eu-short-positions").call(run_input={"countries": [], "includeHistory": false})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/eu-short-positions`.

## Input

| Field | Type | Description |
|---|---|---|
| `countries` | array | Which regulators' registers to read. Leave empty for all nine. |
| `includeHistory` | boolean | Also return historic positions: every earlier update and closed position the registers publish. Most go back to the start of the EU rules in late 2012 (Sweden 2010, Germany 2003), Belgium to 2021, Norway two years. All registers together are about 325,000 positions; combine with ISINs, holders or a from date to keep it small. Default: `false`. |
| `isins` | array | Only positions in these shares, one ISIN per line. Example: `NL0010273215` (ASML). Leave empty for all shares. |
| `holders` | array | Only positions of holders whose name contains one of these words (not case-sensitive). Example: `Citadel`, `Marshall Wace`. |
| `from_date` | string | Only positions dated on or after this day, YYYY-MM-DD. Example: `2026-01-01`. |
| `limit` | integer | Stop after this many positions in total. Leave empty for all. |

## Examples

**All current net short positions in Europe**: Every net short position currently disclosed in the registers of the Netherlands (AFM), France (AMF), the UK (FCA), Germany (BaFin), Italy (CONSOB), Sweden, Norway, Denmark and Belgium: holder, issuer, ISIN, net short % and position date. About 1,700 positions. Export to JSON, CSV or Excel.

```json
{
  "countries": [],
  "includeHistory": false
}
```

**Hedge fund short tracker: Citadel, Marshall Wace, AQR**: Recent short positions of three of the largest short sellers (Citadel, Marshall Wace, AQR) across Europe, current and closed, with the net short % per position date. Schedule it daily to follow their moves. Export to JSON, CSV or Excel.

```json
{
  "countries": [],
  "includeHistory": true,
  "holders": [
    "Citadel",
    "Marshall Wace",
    "AQR"
  ],
  "from_date": "2026-10-01"
}
```

## Output

One dataset item per result. Main fields: `country`, `status`, `isin`, `issuer`, `holder`, `net_short_pct`, `position_date`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable ID, `<country>:<ISIN>:<holder or aggregated>:<position date>` |
| `country` | string | Country of the register (ISO code): NL, FR, GB, NO, SE, DE, IT, DK or BE |
| `regulator` | string | Regulator that publishes the register (AFM, AMF, FCA, Finanstilsynet, Finansinspektionen, BaFin, CONSOB, FSMA) |
| `status` | string | `current` (the holder's latest disclosed position, still at 0.5% or more) or `historic` |
| `isin` | string | ISIN of the shorted share |
| `issuer` | string | Issuer name as the register writes it |
| `issuer_lei` | string | LEI of the issuer (CONSOB only) |
| `holder` | string | Position holder (fund or manager). Null for UK aggregated positions |
| `holder_lei` | string | LEI of the holder (AMF and CONSOB) |
| `aggregated` | boolean | True for the FCA's aggregated net short position per issuer (UK, since July 2026): the sum of all disclosed positions, no holder |
| `net_short_pct` | number | Net short position in % of the issued shares. Null when the register only says `<0.5` |
| `below_threshold` | boolean | True when the position fell below 0.5%: the holder closed or cut it under the disclosure threshold |
| `shares` | integer | Number of shares short (Norway only) |
| `position_date` | string | Date of the position (YYYY-MM-DD) |
| `historic_since` | string | Date the position stopped being published as current, where the register gives it (AMF, FCA, FSMA) |
| `source_url` | string | The register page or file the row comes from |
| `scraped_at` | string | When this run read the register (UTC) |

## Price

Pay per result: $2.00 per 1,000 short positions on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.

Store page, reviews and full documentation: [EU Short Positions Register](https://apify.com/carabain-digital/eu-short-positions?fpr=4ijnom)

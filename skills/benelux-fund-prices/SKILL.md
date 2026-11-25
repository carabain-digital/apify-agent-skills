---
name: benelux-fund-prices
description: "Daily prices (NAV) of 41 Benelux providers' own funds: NL unit-linked policy and PPI funds (NN, a.s.r./Aegon, Brand New Day, Achmea), BE tak 23 (AG, NN, Allianz, AXA, Baloise) and all BEAMA funds. Latest price or full history. Runs the Apify Actor carabain-digital/benelux-fund-prices. Use for tasks like: Dutch insurance and pension fund prices; Belgian branch 23 (tak 23) fund prices."
---

# Benelux Fund Prices

Daily prices (NAV) of 41 Benelux providers' own funds: NL unit-linked policy and PPI funds (NN, a.s.r./Aegon, Brand New Day, Achmea), BE tak 23 (AG, NN, Allianz, AXA, Baloise) and all BEAMA funds. Latest price or full history.

Runs on Apify as the Actor [`carabain-digital/benelux-fund-prices`](https://apify.com/carabain-digital/benelux-fund-prices?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 10,000 fund prices with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~benelux-fund-prices/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~benelux-fund-prices/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"providers": ["nn_nl", "asr", "zwitserleven", "reaal", "allianz_nl", "scildon", "brandnewday", "befrank", "achmea"], "mode": "latest"}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/benelux-fund-prices").call(run_input={"providers": ["nn_nl", "asr", "zwitserleven", "reaal", "allianz_nl", "scildon", "brandnewday", "befrank", "achmea"], "mode": "latest"})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/benelux-fund-prices`.

## Input

| Field | Type | Description |
|---|---|---|
| `providers` | array | Which providers to read. Leave empty for all 41 (about 17,000 funds, 2-3 minutes in latest mode). Example: `nn_nl`, `asr`, `brandnewday`. |
| `funds` | array | Only these funds: an ISIN, the provider's fund code or part of the fund name (not case-sensitive), one per line. Example: `NL0015002PH8` or `Wereld Hedged`. Leave empty for all funds. |
| `product_types` | array | Only these kinds of funds, e.g. `tak23` for every Belgian branch 23 fund. Leave empty for all. |
| `mode` | string | `latest`: one row per fund with its newest price. `history`: every price between From and To date. Providers that publish only the latest price build their history from earlier runs of this Actor (see the history store below), so schedule a daily run for them. Values: `latest`, `history`. Default: `"latest"`. |
| `from_date` | string | History mode: first date, YYYY-MM-DD. Example: `2025-01-01`. Leave empty for everything the provider has (up to 35 years for some funds, so combine with Funds). |
| `to_date` | string | History mode: last date, YYYY-MM-DD. Leave empty for today. |
| `limit` | integer | Stop after this many rows in total. Leave empty for all. |
| `historyStoreName` | string | Named key-value store in your account where the prices of latest-only providers (Zwitserleven, Reaal, AXA, Patronale, ...) are kept between runs. Use the same name in every run (and schedule) to build one history. Example: `benelux-fund-prices-history`. Default: `"benelux-fund-prices-history"`. |

## Examples

**Dutch insurance and pension fund prices**: Latest unit prices of every Dutch unit-linked policy fund and PPI/pension fund: Nationale-Nederlanden (incl. Delta Lloyd, OHRA), a.s.r. (incl. Aegon, Cappital), Zwitserleven, Reaal, Allianz, Scildon, Brand New Day, BeFrank and Achmea. Export to JSON, CSV or Excel.

```json
{
  "providers": [
    "nn_nl",
    "asr",
    "zwitserleven",
    "reaal",
    "allianz_nl",
    "scildon",
    "brandnewday",
    "befrank",
    "achmea"
  ],
  "mode": "latest"
}
```

**Belgian branch 23 (tak 23) fund prices**: Latest unit values of Belgian branch 23 insurance funds from AG Insurance, NN Belgium, Allianz, AXA, Baloise, Patronale Life (Belfius), Argenta, Federale, Ethias and KBC-Life. Export to JSON, CSV or Excel.

```json
{
  "providers": [
    "ag_insurance",
    "nn_be",
    "allianz_be",
    "axa_be",
    "baloise",
    "patronale",
    "argenta_life",
    "federale",
    "ethias",
    "kbc_life"
  ],
  "mode": "latest"
}
```

## Output

One dataset item per result. Main fields: `provider`, `fund_name`, `isin`, `date`, `price`, `currency`, `product_type`, `country`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable ID, `<source>:<fund_id>:<date>` |
| `source` | string | Provider module the row comes from (input value of Providers), e.g. `asr`, `nn_be` |
| `provider` | string | Company that publishes the price (insurer, bank, pension provider or fund manager) |
| `country` | string | Country of the provider: NL, BE or LU |
| `product_type` | string | fund, unit_linked, tak23, ppi, pension, pension_savings or real_estate |
| `fund_id` | string | The provider's own fund code (ISIN when that is its key) |
| `fund_name` | string | Fund name as the provider shows it |
| `isin` | string | ISIN, when the provider publishes one (policy funds often have none) |
| `category` | string | Product line or category on the provider's site |
| `currency` | string | Currency of the price (ISO code) |
| `date` | string | Valuation date of the price, YYYY-MM-DD |
| `price` | number | Unit price / NAV per unit on that date |
| `bid` | number | Bid (sell) price, when the provider publishes one |
| `ask` | number | Ask (buy) price, when the provider publishes one |
| `frequency` | string | How often the provider values the fund: daily, weekly, monthly, quarterly or annual |
| `url` | string | Page or factsheet of the fund |
| `scraped_at` | string | When the row was scraped (UTC, ISO 8601) |

## Price

Pay per result: $0.50 per 1,000 fund prices on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.

Store page, reviews and full documentation: [Benelux Fund Prices](https://apify.com/carabain-digital/benelux-fund-prices?fpr=4ijnom)

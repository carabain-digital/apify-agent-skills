---
name: stock-prices
description: "Daily, weekly and monthly OHLCV price history of shares, ETFs and indices worldwide from Yahoo Finance, by ticker or ISIN: adjusted close, dividends, splits, plus optional RSI, MACD, moving averages, Bollinger bands, ATR, pivot points and support/resistance levels. Runs the Apify Actor carabain-digital/stock-prices. Use for tasks like: US big tech daily prices with RSI, MACD and SMAs; Asian and European blue chips: 5 years of weekly prices."
---

# Stock Price History & Technical Indicators

Daily, weekly and monthly OHLCV price history of shares, ETFs and indices worldwide from Yahoo Finance, by ticker or ISIN: adjusted close, dividends, splits, plus optional RSI, MACD, moving averages, Bollinger bands, ATR, pivot points and support/resistance levels.

Runs on Apify as the Actor [`carabain-digital/stock-prices`](https://apify.com/carabain-digital/stock-prices?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 16,666 price bars with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~stock-prices/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~stock-prices/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"symbols": ["AAPL", "MSFT", "NVDA", "GOOGL", "AMZN", "META", "TSLA"], "interval": "1d", "technicalIndicators": true}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/stock-prices").call(run_input={"symbols": ["AAPL", "MSFT", "NVDA", "GOOGL", "AMZN", "META", "TSLA"], "interval": "1d", "technicalIndicators": true})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/stock-prices`.

## Input

| Field | Type | Description |
|---|---|---|
| `symbols` | array | Yahoo Finance tickers or ISINs, one per line, mixed is fine. Examples: `AAPL`, `MSFT`, `ASML.AS` (Amsterdam), `SAP.DE` (Xetra), `7203.T` (Tokyo), `0700.HK` (Hong Kong), `RELIANCE.NS` (India), `BHP.AX` (Australia), `SHOP.TO` (Toronto), `^GSPC` (S&P 500), `NL0010273215` (ASML by ISIN). An ISIN is mapped to the share's primary listing. Up to 500 per run; symbols Yahoo does not know are skipped and logged. |
| `interval` | string | One row per day (`1d`), week (`1wk`) or month (`1mo`). Example: `1wk` for 5 years of weekly bars. Values: `1d`, `1wk`, `1mo`. Default: `"1d"`. |
| `from_date` | string | First day to return, YYYY-MM-DD. Example: `2020-01-01`. Leave empty for the last 12 months. For the full history use an early date such as `1970-01-01`: Yahoo returns everything it has (large caps go back to the 1980s or earlier). |
| `to_date` | string | Last day to return, YYYY-MM-DD. Example: `2025-12-31`. Leave empty for today. |
| `technicalIndicators` | boolean | Adds SMA 20/50/200, EMA 12/26, RSI 14, MACD (12/26/9), Bollinger bands (20, 2), ATR 14, classic pivot points with R1-R3 / S1-S3, and swing-point support and resistance levels to every row. Computed on the close, with 300 earlier bars as warm-up so the values are valid from the first row. Charged as a separate add-on event per row. Default: `false`. |
| `limit` | integer | Stop after this many rows in total. Example: `5000`. Leave empty for all rows of all symbols. |

## Examples

**US big tech daily prices with RSI, MACD and SMAs**: One year of daily prices for Apple, Microsoft, Nvidia, Alphabet, Amazon, Meta and Tesla with RSI 14, MACD, SMA 20/50/200, Bollinger bands, ATR, pivot points and support/resistance levels on every row. Export to JSON, CSV or Excel.

```json
{
  "symbols": [
    "AAPL",
    "MSFT",
    "NVDA",
    "GOOGL",
    "AMZN",
    "META",
    "TSLA"
  ],
  "interval": "1d",
  "technicalIndicators": true
}
```

**Asian and European blue chips: 5 years of weekly prices**: Five years of weekly OHLCV with adjusted close, dividends and splits for Toyota, Tencent, ASML, SAP, Nestle, LVMH and Samsung, by ticker or ISIN, in the local currency. Handy for backtests and cross-market charts. Export to JSON, CSV or Excel.

```json
{
  "symbols": [
    "7203.T",
    "0700.HK",
    "ASML.AS",
    "SAP.DE",
    "NESN.SW",
    "MC.PA",
    "005930.KS"
  ],
  "interval": "1wk",
  "from_date": "2021-11-01"
}
```

## Output

One dataset item per result. Main fields: `symbol`, `date`, `open`, `high`, `low`, `close`, `adj_close`, `volume`, `change_pct`, `rsi_14`, `sma_50`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable ID, `yahoo:<symbol>:<date>` |
| `symbol` | string | Yahoo Finance symbol the prices belong to (the primary listing when you gave an ISIN) |
| `isin` | string | ISIN you asked for; null when you gave a ticker |
| `name` | string | Company, fund or index name |
| `exchange` | string | Exchange of the listing, as Yahoo names it |
| `currency` | string | Trading currency, ISO code (quotes in pence, cents or agorot are converted: GBp -> GBP, ZAc -> ZAR, ILA -> ILS) |
| `date` | string | Day of the bar in the exchange's local time, YYYY-MM-DD. For weekly and monthly bars the day Yahoo dates the bar |
| `open` | number | Opening price |
| `high` | number | Highest price of the bar |
| `low` | number | Lowest price of the bar |
| `close` | number | Closing price, adjusted for splits but not for dividends |
| `adj_close` | number | Closing price adjusted for splits and dividends (a total-return series). Equals `close` when nothing was paid after the bar |
| `volume` | integer | Shares traded in the bar (0 or null for indices) |
| `dividend` | number | Dividend per share (trading currency) when an ex-date falls in this bar, else null |
| `split_ratio` | string | Split ratio new:old when a split took effect in this bar, e.g. `4:1` or `1:10` (reverse split), else null |
| `change_pct` | number | Close versus the previous bar's close in percent; null for the first bar of the history |
| `scraped_at` | string | When the run collected the bar, UTC ISO 8601 |
| `sma_20` | number | Simple moving average of the last 20 closes. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `sma_50` | number | Simple moving average of the last 50 closes. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `sma_200` | number | Simple moving average of the last 200 closes. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `ema_12` | number | Exponential moving average of the close, 12 bars. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `ema_26` | number | Exponential moving average of the close, 26 bars. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `rsi_14` | number | Relative strength index, 14 bars, Wilder smoothing (0-100). Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `macd` | number | MACD line: EMA 12 minus EMA 26. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `macd_signal` | number | MACD signal line: 9-bar EMA of the MACD line. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `macd_hist` | number | MACD histogram: MACD minus signal. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `bb_upper` | number | Bollinger upper band: SMA 20 plus 2 standard deviations. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `bb_middle` | number | Bollinger middle band: SMA 20. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `bb_lower` | number | Bollinger lower band: SMA 20 minus 2 standard deviations. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `atr_14` | number | Average true range, 14 bars, Wilder smoothing (in price units). Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `pivot` | number | Classic floor pivot point (H+L+C)/3 from the PREVIOUS bar's high, low and close. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `r1` | number | Resistance 1 from the previous bar: 2*pivot - low. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `r2` | number | Resistance 2 from the previous bar: pivot + (high - low). Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `r3` | number | Resistance 3 from the previous bar: high + 2*(pivot - low). Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `s1` | number | Support 1 from the previous bar: 2*pivot - high. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `s2` | number | Support 2 from the previous bar: pivot - (high - low). Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `s3` | number | Support 3 from the previous bar: low - 2*(high - pivot). Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `support` | number | Nearest swing-point support level below the close (cluster mean of 5-bar swing highs and lows of the last 120 bars; only swings confirmed 2 bars earlier are used, so there is no lookahead). Null if none. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `support_touches` | integer | How many swing points form the `support` level. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `resistance` | number | Nearest swing-point resistance level above the close, built like `support`. Null if none. Only with the Technical indicators add-on; null otherwise or when there is not enough history |
| `resistance_touches` | integer | How many swing points form the `resistance` level. Only with the Technical indicators add-on; null otherwise or when there is not enough history |

## Price

Pay per result: $0.30 per 1,000 price bars on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.
- Technical indicators: $0.30 per 1,000. One price bar enriched with SMA 20/50/200, EMA 12/26, RSI 14, MACD, Bollinger bands, ATR 14, pivot points with R1-R3 and S1-S3, and swing-point support and resistance levels. Only bars with at least one value are charged.

Store page, reviews and full documentation: [Stock Price History & Technical Indicators](https://apify.com/carabain-digital/stock-prices?fpr=4ijnom)

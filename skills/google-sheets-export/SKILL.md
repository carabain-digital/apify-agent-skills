---
name: google-sheets-export
description: "Export any Apify dataset to a Google Sheet: replace the tab, append rows, or append only new items by a key field. Works as an integration after any Actor run, on a schedule or through the API. Runs the Apify Actor carabain-digital/google-sheets-export. Use for tasks like: Export a dataset to Google Sheets; Append only new items to a Google Sheet."
---

# Google Sheets Export - Any Dataset to Sheets

Export any Apify dataset to a Google Sheet: replace the tab, append rows, or append only new items by a key field. Works as an integration after any Actor run, on a schedule or through the API.

Runs on Apify as the Actor [`carabain-digital/google-sheets-export`](https://apify.com/carabain-digital/google-sheets-export), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 16,666 exported rows with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~google-sheets-export/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~google-sheets-export/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"datasetId": "MYsSEJZcfvwhcDzhC", "spreadsheet": "https://docs.google.com/spreadsheets/d/1dHj8fy3dQYtPf37NiTl1qj-YqwCzb3jcNLrkpzqIYRI/edit", "sheetName": "Sheet1", "mode": "replace"}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/google-sheets-export").call(run_input={"datasetId": "MYsSEJZcfvwhcDzhC", "spreadsheet": "https://docs.google.com/spreadsheets/d/1dHj8fy3dQYtPf37NiTl1qj-YqwCzb3jcNLrkpzqIYRI/edit", "sheetName": "Sheet1", "mode": "replace"})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/google-sheets-export`.

## Input

| Field | Type | Description |
|---|---|---|
| `datasetId` | string | ID, name or URL of the dataset to export (Storage > Datasets in the Console). As an integration of another Actor (Integrations tab), set it to `{{resource.defaultDatasetId}}` to export that run's results. Example: `WkzbQMuFYuamGv3YF`. |
| `spreadsheet` | string | URL of the spreadsheet, or its ID (the part after `/spreadsheets/d/`). Share it with the service account as Editor first, see the Actor's README. Example: `https://docs.google.com/spreadsheets/d/1AbC.../edit`. |
| `sheetName` | string | The tab to write to. It is created when it does not exist. Example: `Leads`. Default: `"Sheet1"`. |
| `mode` | string | Replace gives you a fresh copy every run. Append keeps the header of the sheet and adds new fields as new columns at the end. Append new skips rows already in the sheet: use it for scheduled runs. Values: `replace`, `append`, `appendNew`. Default: `"replace"`. |
| `keyField` | string | The dataset field that identifies a row, compared with the same column in the sheet. Required for Append new, ignored otherwise. Example: `url` or `id`. |
| `fields` | array | Only these fields become columns, in this order. Use dot paths for nested fields. Empty = all fields, in the order they first appear. Example: `name`, `rating`, `address.city`. |
| `flatten` | boolean | Nested objects become columns named with a dot path (`address.city`), lists of plain values become `a, b`, and lists of objects become a JSON text in one cell. Off = every object or list is a JSON text. Default: `true`. |
| `maxRows` | integer | Export at most this many items, from the start of the dataset. Empty = all. Example: `1000`. |
| `valueInputOption` | string | Raw never turns text into formulas, so scraped text cannot run a formula in your sheet. Choose User entered only for data you trust. Values: `RAW`, `USER_ENTERED`. Default: `"RAW"`. |
| `serviceAccountJson` | string | The whole JSON key of a Google Cloud service account that has the Google Sheets API enabled and edit access to the spreadsheet. Stored encrypted. Empty = the built-in service account. |

## Examples

**Export a dataset to Google Sheets**: Copies every item of an Apify dataset into a tab of your Google Sheet, with a header row and nested data flattened into columns. Replaces the tab on each run, so it is always a fresh copy. Replace the example dataset and spreadsheet with your own.

```json
{
  "datasetId": "MYsSEJZcfvwhcDzhC",
  "spreadsheet": "https://docs.google.com/spreadsheets/d/1dHj8fy3dQYtPf37NiTl1qj-YqwCzb3jcNLrkpzqIYRI/edit",
  "sheetName": "Sheet1",
  "mode": "replace"
}
```

**Append only new items to a Google Sheet**: Adds only the dataset items whose url is not in your Google Sheet yet, so a scheduled scraper builds up a list without duplicates. Change the key field to any unique field of your data. Replace the example dataset and spreadsheet with your own.

```json
{
  "datasetId": "MYsSEJZcfvwhcDzhC",
  "spreadsheet": "https://docs.google.com/spreadsheets/d/1dHj8fy3dQYtPf37NiTl1qj-YqwCzb3jcNLrkpzqIYRI/edit",
  "sheetName": "Items",
  "mode": "appendNew",
  "keyField": "url"
}
```

## Output

One dataset item per result. Main fields: `sheet`, `row`, `exportedAt`.

| Field | Type | Description |
|---|---|---|
| `sheet` | string | Spreadsheet and tab the row was written to |
| `mode` | string | Write mode: replace, append or appendNew |
| `row` | object | The row as written, column name to cell value |
| `exportedAt` | string | Export time, ISO 8601 UTC |

## Price

Pay per result: $0.30 per 1,000 exported rows on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.

Store page, reviews and full documentation: [Google Sheets Export - Any Dataset to Sheets](https://apify.com/carabain-digital/google-sheets-export)

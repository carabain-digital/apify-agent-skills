---
name: website-screenshot-pdf
description: "Full-page screenshots (PNG, JPEG) and real text PDFs of any web page, also behind Cloudflare and other bot walls. Cookie banners and popups removed. Built-in residential proxy fallback. Runs the Apify Actor carabain-digital/website-screenshot-pdf. Use for tasks like: Full-page screenshots of a list of websites; Save web pages as PDF documents."
---

# Website Screenshot & PDF - Full Page, Any Site

Full-page screenshots (PNG, JPEG) and real text PDFs of any web page, also behind Cloudflare and other bot walls. Cookie banners and popups removed. Built-in residential proxy fallback.

Runs on Apify as the Actor [`carabain-digital/website-screenshot-pdf`](https://apify.com/carabain-digital/website-screenshot-pdf?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 1,000 captured pages with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~website-screenshot-pdf/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~website-screenshot-pdf/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"startUrls": [{"url": "https://www.capterra.com/customer-relationship-management-software/"}, {"url": "https://www.bbc.com/news"}, {"url": "https://www.g2.com/categories/crm"}], "format": "png", "fullPage": true}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/website-screenshot-pdf").call(run_input={"startUrls": [{"url": "https://www.capterra.com/customer-relationship-management-software/"}, {"url": "https://www.bbc.com/news"}, {"url": "https://www.g2.com/categories/crm"}], "format": "png", "fullPage": true})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/website-screenshot-pdf`.

## Input

| Field | Type | Description |
|---|---|---|
| `startUrls` | array | The pages to capture, one file per page. Paste a list, upload a text or CSV file, or link a file with URLs. `example.com` without https:// also works. Format: `[{"url": "https://..."}]`. |
| `format` | string | Screenshot as PNG or JPEG, or a PDF printed by the browser with real, selectable text and page breaks. Values: `png`, `jpeg`, `pdf`. Default: `"png"`. |
| `fullPage` | boolean | Capture the whole page down to the footer (lazy images are loaded first). Off = only the first screen. Very long pages are cut at 30,000 pixels. Screenshots only. Default: `true`. |
| `device` | string | Screen size the page is rendered at. Screenshots only; a PDF is laid out for its paper size. Values: `desktop`, `laptop`, `tablet`, `mobile`. Default: `"desktop"`. |
| `viewportWidth` | integer | Overrides the device width. Example: `1440`. |
| `viewportHeight` | integer | Overrides the device height (the first screen when Full page is off). Example: `900`. |
| `jpegQuality` | integer | Lower = smaller files. JPEG only. Default: `80`. |
| `pdfFormat` | string | Paper size of the PDF. Values: `A4`, `Letter`, `Legal`, `A3`, `A5`. Default: `"A4"`. |
| `pdfLandscape` | boolean | Print the PDF in landscape orientation. Default: `false`. |
| `pdfMarginInches` | number | Page margin on all sides. Example: `0` for edge to edge. Default: `0.4`. |
| `pdfBackground` | boolean | Keep background colours and images, as on screen. Default: `true`. |
| `dismissCookieBanners` | boolean | Click the "Accept all" button of cookie banners (OneTrust, Cookiebot, Didomi, Usercentrics and others, in many languages) and hide what remains. Default: `true`. |
| `hidePopups` | boolean | Close newsletter modals, sign-in prompts and dimmed overlays that cover the page. Small fixed headers and chat buttons stay. Default: `true`. |
| `hideSelectors` | array | Extra elements to hide before capturing. Example: `.ad-banner`, `#chat-widget`. |
| `waitForSelector` | string | Wait until this element appears (up to 30 seconds), for pages that load content late. Example: `.results-table`. |
| `waitSeconds` | number | Pause before capturing, after the page has loaded and banners are closed. Raise it for animations or charts. Default: `1`. |
| `proxyCountry` | string | Load every page through a residential IP in this country, to capture the local version of a site (language, prices, geo-blocked pages). Each page through the residential IP costs the Unblocked page add-on, $5.00 per 1,000. Empty = pages load directly, and only a page that blocks the run is retried through a US residential IP automatically (and charged as Unblocked page when that works). Values: `US`, `GB`, `DE`, `FR`, `NL`, `BE`, `ES`, `IT`, `CA`, `AU`, `JP`, `BR`, `IN`. |
| `maxItems` | integer | Capture at most this many of the URLs. Example: `100`. |

## Examples

**Full-page screenshots of a list of websites**: A full-page PNG of every URL in the list, with cookie banners and popups removed, also for sites behind Cloudflare. Paste your own URLs. Download the images from the Output tab or by API.

```json
{
  "startUrls": [
    {
      "url": "https://www.capterra.com/customer-relationship-management-software/"
    },
    {
      "url": "https://www.bbc.com/news"
    },
    {
      "url": "https://www.g2.com/categories/crm"
    }
  ],
  "format": "png",
  "fullPage": true
}
```

**Save web pages as PDF documents**: Every URL printed to an A4 PDF with selectable text and page breaks, backgrounds kept and cookie banners removed. For archiving, compliance records and offline reading. Paste your own URLs.

```json
{
  "startUrls": [
    {
      "url": "https://en.wikipedia.org/wiki/Web_scraping"
    },
    {
      "url": "https://www.capterra.com/p/155928/Zoho-CRM/"
    }
  ],
  "format": "pdf",
  "pdfFormat": "A4"
}
```

## Output

One dataset item per result. Main fields: `fileUrl`, `url`, `title`, `statusCode`, `format`, `fileSizeBytes`, `viaResidentialProxy`.

| Field | Type | Description |
|---|---|---|
| `url` | string | Requested page URL |
| `finalUrl` | string | URL after redirects |
| `title` | string | Page title |
| `statusCode` | integer | HTTP status of the page |
| `format` | string | png, jpeg or pdf |
| `fileUrl` | string | Download link of the screenshot or PDF (run's key-value store) |
| `fileKey` | string | Key of the file in the run's key-value store |
| `fileSizeBytes` | integer | File size in bytes |
| `viewport` | string | Browser viewport, width x height in CSS pixels |
| `fullPage` | boolean | Whole page (true) or the first screen (false); null for PDFs |
| `viaResidentialProxy` | boolean | Captured through the residential proxy |
| `capturedAt` | string | Capture time, ISO 8601 UTC |

## Price

Pay per result: $5.00 per 1,000 captured pages on the Apify Free plan, less on paid plans, plus $0.01 per run start per GB of memory. No subscription; Apify bills the run to the user's account.
- Unblocked page (residential IP): $5.00 per 1,000. One page that was loaded through a residential IP: a page that blocked the direct load (Cloudflare, DataDome, HTTP 403/429), or every page with Capture from country. Charged only when the capture succeeded.

Store page, reviews and full documentation: [Website Screenshot & PDF - Full Page, Any Site](https://apify.com/carabain-digital/website-screenshot-pdf?fpr=4ijnom)

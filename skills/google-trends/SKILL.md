---
name: google-trends
description: "Google Trends data by search term or URL: interest over time, by country, region, city and metro, related queries, plus Trending now per country. Same output as Apify's Google Trends Scraper, with a residential IP per term so runs don't hit 429. Runs the Apify Actor carabain-digital/google-trends. Use for tasks like: Keyword trends in the US: past 12 months; Trending now: US, UK, Germany and the Netherlands."
---

# Google Trends Scraper

Google Trends data by search term or URL: interest over time, by country, region, city and metro, related queries, plus Trending now per country. Same output as Apify's Google Trends Scraper, with a residential IP per term so runs don't hit 429.

Runs on Apify as the Actor [`carabain-digital/google-trends`](https://apify.com/carabain-digital/google-trends), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 2,631 results with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~google-trends/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~google-trends/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"searchTerms": ["web scraping", "artificial intelligence", "chatgpt"], "geo": "US", "timeRange": ""}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/google-trends").call(run_input={"searchTerms": ["web scraping", "artificial intelligence", "chatgpt"], "geo": "US", "timeRange": ""})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/google-trends`.

## Input

| Field | Type | Description |
|---|---|---|
| `searchTerms` | array | Search terms to look up, one per line. Each term gives one result. Turn on **Multiple terms** to compare up to 5 comma-separated terms in one result, like on trends.google.com. |
| `isMultiple` | boolean | If checked, a comma separates terms that are compared with each other (`bitcoin, ethereum`), up to 5 per line. Default: `false`. |
| `timeRange` | string | Predefined time range (default: past 12 months). Values: ``, `now 1-H`, `now 4-H`, `now 1-d`, `now 7-d`, `today 1-m`, `today 3-m`, `today 5-y`, `all`. Default: `""`. |
| `customTimeRange` | string | Overrides Time range. Format `YYYY-MM-DD YYYY-MM-DD`, e.g. `2024-01-01 2024-06-30`. |
| `geo` | string | Country to get results for (default: worldwide). Worldwide gives interest by country; a country gives interest by subregion and city, the US also by metro area (DMA). Values: ``, `AF`, `AL`, `DZ`, `AS`, `AD`, `AO`, `AI`, `AQ`, `AG`, `AR`, `AM`, `AW`, `AU`, `AT`, `AZ`, `BS`, `BH`, `BD`, `BB`, `BY`, `BE`, `BZ`, `BJ`, `BM`, `BT`, `BO`, `BA`, `BW`, `BV`, `BR`, `IO`, `VG`, `BN`, `BG`, `BF`, `BI`, `KH`, `CM`, `CA`, `CV`, `BQ`, `KY`, `CF`, `TD`, `CL`, `CN`, `CX`, `CC`, `CO`, `KM`, `CG`, `CD`, `CK`, `CR`, `HR`, `CU`, `CW`, `CY`, `CZ`, `CI`, `DK`, `DJ`, `DM`, `DO`, `EC`, `EG`, `SV`, `GQ`, `ER`, `EE`, `SZ`, `ET`, `FK`, `FO`, `FJ`, `FI`, `FR`, `GF`, `PF`, `TF`, `GA`, `GM`, `GE`, `DE`, `GH`, `GI`, `GR`, `GL`, `GD`, `GP`, `GU`, `GT`, `GG`, `GN`, `GW`, `GY`, `HT`, `HM`, `HN`, `HK`, `HU`, `IS`, `IN`, `ID`, `IR`, `IQ`, `IE`, `IM`, `IL`, `IT`, `JM`, `JP`, `JE`, `JO`, `KZ`, `KE`, `KI`, `XK`, `KW`, `KG`, `LA`, `LV`, `LB`, `LS`, `LR`, `LY`, `LI`, `LT`, `LU`, `MO`, `MG`, `MW`, `MY`, `MV`, `ML`, `MT`, `MH`, `MQ`, `MR`, `MU`, `YT`, `MX`, `FM`, `MD`, `MC`, `MN`, `ME`, `MS`, `MA`, `MZ`, `MM`, `NA`, `NR`, `NP`, `NL`, `NC`, `NZ`, `NI`, `NE`, `NG`, `NU`, `NF`, `KP`, `MK`, `MP`, `NO`, `OM`, `PK`, `PW`, `PS`, `PA`, `PG`, `PY`, `PE`, `PH`, `PN`, `PL`, `PT`, `PR`, `QA`, `RO`, `RU`, `RW`, `RE`, `WS`, `SM`, `SA`, `SN`, `RS`, `SC`, `SL`, `SG`, `SX`, `SK`, `SI`, `SB`, `SO`, `ZA`, `GS`, `KR`, `SS`, `ES`, `LK`, `BL`, `SH`, `KN`, `LC`, `MF`, `PM`, `VC`, `SD`, `SR`, `SJ`, `SE`, `CH`, `SY`, `ST`, `TW`, `TJ`, `TZ`, `TH`, `TL`, `TG`, `TK`, `TO`, `TT`, `TN`, `TM`, `TC`, `TV`, `TR`, `UM`, `VI`, `UG`, `UA`, `AE`, `GB`, `US`, `UY`, `UZ`, `VU`, `VA`, `VE`, `VN`, `WF`, `EH`, `YE`, `ZM`, `ZW`, `AX`. Default: `""`. |
| `category` | string | Limit the search to a category (default: all categories). Values: ``, `3`, `47`, `44`, `22`, `12`, `5`, `7`, `71`, `8`, `45`, `65`, `11`, `13`, `958`, `19`, `16`, `299`, `14`, `66`, `29`, `533`, `174`, `18`, `20`, `67`. Default: `""`. |
| `property` | string | Which Google search to measure (default: web search). Values: ``, `images`, `news`, `froogle`, `youtube`. Default: `""`. |
| `startUrls` | array | Paste explore URLs from trends.google.com, e.g. `https://trends.google.com/trends/explore?q=bitcoin,ethereum&geo=US`. Terms, geo, date, category and property are read from the URL. Format: `[{"url": "https://..."}]`. |
| `spreadsheetId` | string | (Optional) ID of a public Google Sheet with search terms in the first column; row 1 is the header. |
| `trendingCountries` | array | Also get the Trending now list (what people search for right now) for these countries, two-letter codes such as `US`, `NL`, `DE`. Up to about 200 trends per country with search volume, increase %, related searches and categories. |
| `trendingHours` | string | Time window for Trending now. Values: `4`, `24`, `48`, `168`. Default: `"24"`. |
| `viewedFrom` | string | Country of the exit IP. Google Trends numbers do not depend on it; leave empty unless you need a specific country. Values: ``, `af`, `al`, `dz`, `as`, `ad`, `ao`, `ai`, `aq`, `ag`, `ar`, `am`, `aw`, `au`, `at`, `az`, `bs`, `bh`, `bd`, `bb`, `by`, `be`, `bz`, `bj`, `bm`, `bt`, `bo`, `ba`, `bw`, `bv`, `br`, `io`, `vg`, `bn`, `bg`, `bf`, `bi`, `kh`, `cm`, `ca`, `cv`, `bq`, `ky`, `cf`, `td`, `cl`, `cn`, `cx`, `cc`, `co`, `km`, `cg`, `cd`, `ck`, `cr`, `hr`, `cu`, `cw`, `cy`, `cz`, `ci`, `dk`, `dj`, `dm`, `do`, `ec`, `eg`, `sv`, `gq`, `er`, `ee`, `sz`, `et`, `fk`, `fo`, `fj`, `fi`, `fr`, `gf`, `pf`, `tf`, `ga`, `gm`, `ge`, `de`, `gh`, `gi`, `gr`, `gl`, `gd`, `gp`, `gu`, `gt`, `gg`, `gn`, `gw`, `gy`, `ht`, `hm`, `hn`, `hk`, `hu`, `is`, `in`, `id`, `ir`, `iq`, `ie`, `im`, `il`, `it`, `jm`, `jp`, `je`, `jo`, `kz`, `ke`, `ki`, `xk`, `kw`, `kg`, `la`, `lv`, `lb`, `ls`, `lr`, `ly`, `li`, `lt`, `lu`, `mo`, `mg`, `mw`, `my`, `mv`, `ml`, `mt`, `mh`, `mq`, `mr`, `mu`, `yt`, `mx`, `fm`, `md`, `mc`, `mn`, `me`, `ms`, `ma`, `mz`, `mm`, `na`, `nr`, `np`, `nl`, `nc`, `nz`, `ni`, `ne`, `ng`, `nu`, `nf`, `kp`, `mk`, `mp`, `no`, `om`, `pk`, `pw`, `ps`, `pa`, `pg`, `py`, `pe`, `ph`, `pn`, `pl`, `pt`, `pr`, `qa`, `ro`, `ru`, `rw`, `re`, `ws`, `sm`, `sa`, `sn`, `rs`, `sc`, `sl`, `sg`, `sx`, `sk`, `si`, `sb`, `so`, `za`, `gs`, `kr`, `ss`, `es`, `lk`, `bl`, `sh`, `kn`, `lc`, `mf`, `pm`, `vc`, `sd`, `sr`, `sj`, `se`, `ch`, `sy`, `st`, `tw`, `tj`, `tz`, `th`, `tl`, `tg`, `tk`, `to`, `tt`, `tn`, `tm`, `tc`, `tv`, `tr`, `um`, `vi`, `ug`, `ua`, `ae`, `gb`, `us`, `uy`, `uz`, `vu`, `va`, `ve`, `vn`, `wf`, `eh`, `ye`, `zm`, `zw`, `ax`. |
| `language` | string | Language of category and topic names, e.g. `en-US`, `nl`, `de`. Default: `"en-US"`. |
| `maxItems` | integer | Maximum number of results. Zero means no limit. Default: `0`. |

## Examples

**Keyword trends in the US: past 12 months**: Google Trends for a list of keywords in the United States over the past 12 months: weekly interest over time, interest by state, city and metro area, and top and rising related queries. Schedule it monthly to track SEO and content demand. Export to JSON, CSV or Excel.

```json
{
  "searchTerms": [
    "web scraping",
    "artificial intelligence",
    "chatgpt"
  ],
  "geo": "US",
  "timeRange": ""
}
```

**Trending now: US, UK, Germany and the Netherlands**: What people search for right now on Google in the US, UK, Germany and the Netherlands: up to 200 trending searches per country with search volume, increase %, related searches and category. Schedule it hourly or daily for news, content and ad ideas. Export to JSON, CSV or Excel.

```json
{
  "trendingCountries": [
    "US",
    "GB",
    "DE",
    "NL"
  ],
  "trendingHours": "24"
}
```

## Output

One dataset item per result. Main fields: `searchTerm`, `geo`, `timeRange`, `interestOverTime_timelineData`, `relatedQueries_top`, `relatedQueries_rising`.

| Field | Type | Description |
|---|---|---|
| `inputUrlOrTerm` | string | The search term or Google Trends URL from the input |
| `searchTerm` | string | The term, or the compared terms joined by `, ` |
| `geo` | string | Geo area (`Worldwide` or a country/region code) |
| `timeRange` | string | Time range as Google writes it (`today 12-m`, `2024-01-01 2024-06-30`) |
| `category` | string | Category id (0 = all) |
| `property` | string | web, images, news, froogle (Shopping) or youtube |
| `interestOverTime_timelineData` | array | Interest over time: per point `time` (epoch), `date` (ISO), `formattedTime`, `value` (0-100, one per term), `hasData`, `formattedValue`, `isPartial` |
| `interestOverTime_averages` | array | Average interest per term (only when terms are compared) |
| `interestBySubregion` | array | Interest by country (worldwide) or by region of the country: `geoCode`, `geoName`, `value`, `formattedValue`, `maxValueIndex`, `hasData` |
| `interestByCity` | array | Interest by city with `coordinates` (country searches only) |
| `interestByMetro` | array | Interest by US metro area (DMA), US only |
| `relatedQueries_top` | array | Top related queries: `query`, `value`, `formattedValue`, `link` (and `searchTerm` when terms are compared) |
| `relatedQueries_rising` | array | Rising related queries; `formattedValue` is the increase (`+250%`, `Breakout`) |
| `relatedTopics_top` | array | Top related topics (`topic: {mid, title, type}`); Google withholds these from automated access, so usually empty |
| `relatedTopics_rising` | array | Rising related topics; usually empty, see above |
| `type` | string | `trending` for Trending now items (search results have no type) |
| `query` | string | Trending now: the trending search |
| `searchVolume` | integer | Trending now: approximate searches (lower bound, e.g. 50000 = 50K+) |
| `increasePercentage` | integer | Trending now: increase in % |
| `active` | boolean | Trending now: still trending |
| `startedAt` | string | Trending now: when the trend started (UTC) |
| `endedAt` | string | Trending now: when it stopped trending (UTC) |
| `trendBreakdown` | array | Trending now: related searches grouped into this trend |
| `categories` | array | Trending now: categories (Sports, Entertainment, ...) |
| `url` | string | Trending now: Google Trends explore link |
| `scrapedAt` | string | When the data was read (UTC) |

## Price

Pay per result: $1.90 per 1,000 results on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.

Store page, reviews and full documentation: [Google Trends Scraper](https://apify.com/carabain-digital/google-trends)

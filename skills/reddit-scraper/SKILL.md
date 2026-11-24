---
name: reddit-scraper
description: "Scrape Reddit posts, comments, subreddits, users and search results without login or API key: title, text, score, upvote ratio, comments with replies, flair, media. Same output as Reddit Scraper Lite, with a built-in residential proxy. Runs the Apify Actor carabain-digital/reddit-scraper. Use for tasks like: Reddit brand monitoring: new posts about a keyword; Top posts of a subreddit this week with comments."
---

# Reddit Scraper - Posts, Comments, Search

Scrape Reddit posts, comments, subreddits, users and search results without login or API key: title, text, score, upvote ratio, comments with replies, flair, media. Same output as Reddit Scraper Lite, with a built-in residential proxy.

Runs on Apify as the Actor [`carabain-digital/reddit-scraper`](https://apify.com/carabain-digital/reddit-scraper?fpr=4ijnom), billed per result to the user's own Apify account.

## Setup

The user needs an Apify API token in `APIFY_TOKEN` ([console.apify.com/settings/integrations](https://console.apify.com/settings/integrations)). No Apify account yet? [Sign up free](https://console.apify.com/sign-up?fpr=4ijnom): no credit card needed, and the Free plan includes $5 of credit every month, enough for about 5,555 items with this Actor.

## Run it

Apify API, waits for the run and returns the items (up to 5 minutes; for bigger runs start it with `POST /v2/acts/carabain-digital~reddit-scraper/runs` and read the dataset afterwards):

```bash
curl -X POST 'https://api.apify.com/v2/acts/carabain-digital~reddit-scraper/run-sync-get-dataset-items' \
  -H "Authorization: Bearer $APIFY_TOKEN" -H 'Content-Type: application/json' \
  -d '{"searches": ["apify"], "sort": "new", "maxPostCount": 25, "maxComments": 5, "maxItems": 200}'
```

Python (`pip install apify-client`):

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])
run = client.actor("carabain-digital/reddit-scraper").call(run_input={"searches": ["apify"], "sort": "new", "maxPostCount": 25, "maxComments": 5, "maxItems": 200})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

MCP: add the Apify MCP server with this Actor as a tool: `https://mcp.apify.com/?actors=carabain-digital/reddit-scraper`.

## Input

| Field | Type | Description |
|---|---|---|
| `startUrls` | array | Subreddits (`/r/python/`, `/r/python/new/`, `/r/python/top/?t=week`), posts (`/r/.../comments/<id>/...`), users (`/user/<name>/`) or search pages (`/search/?q=...`). Format: `[{"url": "https://..."}]`. |
| `searches` | array | Search Reddit for these terms (posts, and communities if enabled below). One term per line. |
| `searchCommunityName` | string | Only search inside this subreddit, e.g. `programming`. |
| `searchPosts` | boolean | Return posts that match the search terms. Default: `true`. |
| `searchCommunities` | boolean | Also return communities (subreddits) that match the search terms. Default: `false`. |
| `sort` | string | Order of search results. Values: `relevance`, `hot`, `top`, `new`, `comments`. Default: `"new"`. |
| `time` | string | For search and for top subreddit feeds. Values: `all`, `hour`, `day`, `week`, `month`, `year`. Default: `"all"`. |
| `maxItems` | integer | Maximum items (posts, comments and communities together) in the dataset. 0 = no limit. Default: `10`. |
| `maxPostCount` | integer | Posts per subreddit, user, search term or search URL. Default: `10`. |
| `maxComments` | integer | Comments per post (in reading order, replies included). 0 = no comments. Default: `10`. |
| `maxCommunitiesCount` | integer | Communities per search term when Search for communities is on. Default: `2`. |
| `skipComments` | boolean | Don't scrape comments (faster: posts come straight from the feed). Default: `false`. |
| `skipUserPosts` | boolean | Ignore user profile URLs. Default: `false`. |
| `skipCommunity` | boolean | Don't add a community item (name, description, weekly visitors) for subreddit URLs. Default: `false`. |
| `includeNSFW` | boolean | Include posts marked NSFW (18+). Default: `true`. |
| `postDateLimit` | string | Only posts created on or after this day, YYYY-MM-DD. |
| `commentDateLimit` | string | Only comments created on or after this day, YYYY-MM-DD. |
| `ignoreStartUrls` | boolean | Ignore Reddit URLs and only run the searches (for Zapier/Make setups). Default: `false`. |

## Examples

**Reddit brand monitoring: new posts about a keyword**: Newest Reddit posts that mention your brand or keyword, with text, score, subreddit and the top comments, for social listening and lead finding. Schedule it daily and compare on id. Export to JSON, CSV or Excel.

```json
{
  "searches": [
    "apify"
  ],
  "sort": "new",
  "maxPostCount": 25,
  "maxComments": 5,
  "maxItems": 200
}
```

**Top posts of a subreddit this week with comments**: The top posts of r/dataisbeautiful of the past week with their text, score, upvote ratio and the first 20 comments with replies. Swap the subreddit for any community. Export to JSON, CSV or Excel.

```json
{
  "startUrls": [
    {
      "url": "https://www.reddit.com/r/dataisbeautiful/top/?t=week"
    }
  ],
  "maxPostCount": 10,
  "maxComments": 20,
  "maxItems": 300
}
```

## Output

One dataset item per result. Main fields: `dataType`, `username`, `title`, `body`, `upVotes`, `numberOfComments`, `communityName`, `createdAt`, `url`.

| Field | Type | Description |
|---|---|---|
| `dataType` | string | `post`, `comment` or `community` |
| `id` | string | Reddit fullname: `t3_` post, `t1_` comment, `t5_` community |
| `parsedId` | string | ID without the prefix |
| `url` | string | Reddit URL of the item |
| `username` | string | Author |
| `userId` | string | Author ID (`t2_`), posts only |
| `authorFlair` | string | Author flair (not shown to logged-out visitors, always null) |
| `title` | string | Post title, or community name |
| `communityName` | string | Subreddit with `r/` |
| `parsedCommunityName` | string | Subreddit without `r/` |
| `category` | string | Subreddit |
| `body` | string | Text of the post or comment (community: description) |
| `html` | string | HTML of the text |
| `link` | string | Link of the post (external URL, image or the post) |
| `postId` | string | Comments: the post (`t3_`) |
| `parentId` | string | Comments: the parent comment (`t1_`) or the post |
| `numberOfComments` | integer | Posts: number of comments |
| `numberOfReplies` | integer | Comments: direct replies among the scraped comments |
| `flair` | string | Post flair |
| `upVotes` | integer | Score (upvotes minus downvotes) |
| `upVoteRatio` | number | Share of upvotes (posts from feeds and post pages) |
| `isVideo` | boolean | Video post |
| `isAd` | boolean | Promoted post |
| `over18` | boolean | NSFW |
| `videoUrls` | array | Video URL |
| `thumbnailUrl` | string | Preview image |
| `imageUrls` | array | Image URLs (image and gallery posts) |
| `postType` | string | text, link, image, gallery, video, ... |
| `depth` | integer | Comments: 0 = top level |
| `weeklyVisitors` | integer | Communities: weekly visitors |
| `weeklyContributions` | integer | Communities: weekly posts and comments |
| `createdAt` | string | Created (ISO, UTC) |
| `scrapedAt` | string | Scraped (ISO, UTC) |

## Price

Pay per result: $0.90 per 1,000 items on the Apify Free plan, less on paid plans. No subscription; Apify bills the run to the user's account.

Store page, reviews and full documentation: [Reddit Scraper - Posts, Comments, Search](https://apify.com/carabain-digital/reddit-scraper?fpr=4ijnom)

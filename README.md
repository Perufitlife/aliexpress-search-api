# AliExpress search API: products, prices, units sold and ratings as JSON

How to get AliExpress search results for a keyword as JSON, without an AliExpress developer app, an affiliate account or a login: product ID, price, original price, discount, star rating, units sold, ship-from country, delivery, store and images, for the ship-to country and currency you choose.

The data comes from the Apify Actor [AliExpress Search Scraper](https://apify.com/renzomacar/aliexpress-search-scraper). It reads the public search pages, the same ones a visitor sees without signing in. No proxy to set up.

## Why not the official API

AliExpress' Open Platform and affiliate APIs need a registered app and an approved key, and they are built around affiliate links and your own orders. They do not give you the search results page as a shopper sees it: the order products appear in for a keyword, which cards are sponsored, the "10K+ sold" counter, or the price shown to a buyer in a given country.

## Example: best-selling yoga mats, shipped to the US

Input:

```json
{
  "queries": ["yoga mat"],
  "maxResults": 30,
  "sortBy": "orders",
  "includeAds": false
}
```

Run on 2026-11-21: 30 products in 3.9 seconds. First twelve (full file: [sample-output.json](sample-output.json)):

| # | Product ID | Title | Price (USD) | Original | Discount | Rating | Sold | Ships from |
|---|---|---|---|---|---|---|---|---|
| 1 | 1005006995272203 | Yoga Mat Pilates Fitness Mat 3/4/6mm Thicknes… | 5.19 | 12.74 | 59% | 4.9 | 10K+ | CN |
| 2 | 1005007567028619 | Yoga Kneeling Mat Thickened Flat Support Mat … | 3.39 | 13.23 | 74% | 4.7 | 8K+ | CN |
| 3 | 1005005828306541 | 4MM Thick EVA Yoga Mats Anti-slip Sport Fitne… | 1.09 | 23.29 | 95% | 4.4 | 6K+ | CN |
| 4 | 1005006828003763 | Yoga Mat Sports Fitness EVA Comfortable Foam … | 1.09 | 17.21 | 93% | 4.5 | 5K+ | CN |
| 5 | 1005008660354269 | 4mm Thick TPE Foldable Yoga Mat Easy to Store… | 11.89 | 25.04 | 52% | 4.9 | 4K+ | CN |
| 6 | 1005006522319965 | 30*30cm Thicken Sports Gym Mat Protection EVA… | 7.2 | 50.15 | 85% | 4.6 | 3K+ | CN |
| 7 | 1005010759550315 | 30x30cm 16 Pcs Interlocking Yoga Mat Home Gym… | 8.72 | 38.09 | 77% | 4.9 | 3K+ | CN |
| 8 | 1005012014054404 | Balance Training Pad Non-Slip High Rebound Th… | 1.09 | 10.66 | 89% | 4.9 | 3K+ | CN |
| 9 | 1005010109568797 | 8/16 Pcs Foam Fitness Non Slip Yoga Mat Fitne… | 9.75 | 20.11 | 51% | 4.9 | 2K+ | CN |
| 10 | 1005008144967169 | Balance Pad, Non-Slip Foam Mat & Ankles Knee … | 12.99 | 27.34 | 52% | 4.9 | 2K+ | CN |
| 11 | 1005008406100897 | Anti-slip Yoga MATS, Pilates Fitness MATS, Ec… | 7.92 | 35.54 | 77% | 4.9 | 2K+ | CN |
| 12 | 1005008430151207 | 8mm NBR Yoga Mat Anti Slip Thickened Style fo… | 6.04 | 33.27 | 81% | 4.9 | 2K+ | CN |

Sorted by orders, the "sold" counter goes down row by row, which makes it a quick way to see what actually sells in a niche and at what price.

One record:

```json
{
  "query": "yoga mat",
  "searchUrl": null,
  "page": 1,
  "position": 1,
  "productId": "1005006995272203",
  "url": "https://www.aliexpress.com/item/1005006995272203.html",
  "title": "Yoga Mat Pilates Fitness Mat 3/4/6mm Thicknes Non Slip Yoga Cushion Travel Fitness Exercise Pad for Women Home Gym Floor Workout",
  "price": 5.19,
  "originalPrice": 12.74,
  "currency": "USD",
  "discountPercent": 59,
  "priceText": "US $5.19",
  "rating": 4.9,
  "sold": 10000,
  "soldText": "10K+ sold",
  "shipFrom": "CN",
  "freeShipping": false,
  "deliveryText": null,
  "storeName": null,
  "sellerCompany": null,
  "badges": [],
  "imageUrl": "https://ae-pic-a1.aliexpress-media.com/kf/S2519edadb8104a8a914b2c6c1ff4151bZ.jpg",
  "images": [
    "https://ae-pic-a1.aliexpress-media.com/kf/S2519edadb8104a8a914b2c6c1ff4151bZ.jpg",
    "https://ae-pic-a1.aliexpress-media.com/kf/S098ef04cfd1c4366b55c986374788dfaS.jpg"
  ],
  "isAd": false,
  "newShopperDeal": false,
  "listedAt": "2024-05-16 00:00:00",
  "shipTo": "US",
  "scrapedAt": "2026-11-21T23:20:47.117Z"
}
```

`sold` is the lower bound AliExpress prints ("10K+ sold" becomes 10000). `productId` is the one in the product page URL, so it joins with product-detail and review tools. `storeName`, `deliveryText` and `badges` are filled only when AliExpress prints them on the search card, which varies from page to page.

## Call it from code

You need an Apify account and its API token (`APIFY_TOKEN`).

curl, results back in the same request:

```bash
curl -X POST "https://api.apify.com/v2/acts/renzomacar~aliexpress-search-scraper/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "content-type: application/json" \
  -d '{"queries":["yoga mat"],"maxResults":30,"sortBy":"orders","includeAds":false}'
```

Python, a quick niche check: how many sellers pass 1,000 sales and at what price:

```python
from statistics import median
from apify_client import ApifyClient

client = ApifyClient("YOUR_APIFY_TOKEN")
run = client.actor("renzomacar/aliexpress-search-scraper").call(run_input={
    "queries": ["posture corrector", "neck massager", "led strip lights"],
    "maxResults": 120,
    "sortBy": "orders",
    "minSold": 1000,
    "includeAds": False,
})
by_query = {}
for p in client.dataset(run["defaultDatasetId"]).iterate_items():
    by_query.setdefault(p["query"], []).append(p["price"])
for q, prices in by_query.items():
    print(f"{q}: {len(prices)} products with 1,000+ sold, median price {median(prices):.2f}")
```

JavaScript, the same search priced for three countries:

```js
import { ApifyClient } from "apify-client";

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });
for (const [shipTo, currency] of [["US", "USD"], ["GB", "GBP"], ["DE", "EUR"]]) {
  const run = await client.actor("renzomacar/aliexpress-search-scraper").call({
    queries: ["wireless earbuds"],
    maxResults: 60,
    sortBy: "orders",
    shipTo,
    currency,
  });
  const { items } = await client.dataset(run.defaultDatasetId).listItems();
  console.log(shipTo, items.length, items[0]?.title.slice(0, 50), items[0]?.price, items[0]?.currency);
}
```

## Use it from Claude, ChatGPT or another AI agent (MCP)

The Actor is available as a tool through the Apify MCP server. Add this to your MCP client configuration:

```json
{
  "mcpServers": {
    "apify": {
      "url": "https://mcp.apify.com/?tools=renzomacar/aliexpress-search-scraper",
      "headers": { "Authorization": "Bearer YOUR_APIFY_TOKEN" }
    }
  }
}
```

Then ask in plain words, for example: "Find the 20 best-selling yoga mats on AliExpress shipped to the UK, in pounds, rated 4.5 or more, and tell me the price range."

## Input fields

| Field | What it does |
|---|---|
| `queries` | Search terms, each searched separately. |
| `startUrls` | AliExpress search or category pages copied from the browser, with their own filters. |
| `maxResults` | Products per search, up to 3,000. AliExpress shows 60 per page. |
| `sortBy` | `best_match`, `orders`, `price_asc`, `price_desc`. |
| `shipTo`, `currency` | Destination country and currency, for example `GB` + `GBP`, `DE` + `EUR`, `BR` + `BRL`. Prices and delivery change by destination. |
| `shipFrom` | Only products sent from a warehouse in that country (`US`, `ES`, `FR`, `CN`...). |
| `minPrice`, `maxPrice` | Price range in the chosen currency. |
| `minRating`, `minSold` | Keep only products rated at least this, or with at least this many sold. |
| `includeAds` | Turn off for organic results only. |

## What it does not do

- Search pages only. Variants, specifications, full description, shipping cost per option and reviews live on the product page and are not included.
- Prices are the ones shown to a visitor who is not signed in. Products marked `newShopperDeal: true` show AliExpress' welcome price for first-time buyers; the regular price can be higher.
- AliExpress rounds large sales counters (1,000+, 10K+), so `sold` is a lower bound, not an exact count.

## Price

Pay per result on Apify, with the price shown on the Actor page. Products left out by your filters (sponsored, rating, units sold) are not charged, and you can set a maximum cost per run.

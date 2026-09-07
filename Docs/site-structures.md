# Site Structure Notes

**Purpose:** A scraper can't "see" a page the way a person can — no layout,
no visual grid, nothing obviously labelled "this is the price." Before any
spider can be written, someone has to manually feel out how a retailer's
site actually stores its data. That discovery work is expensive and easy
to lose if it only lives in one person's memory (or one Claude
conversation). This file is where the findings get kept, so the next
store — or the next person entirely — can skip straight to writing the
spider instead of rediscovering the page structure from scratch.

See MASTER.md Section 9 for *why* this discovery process looks the way it
does. This file is just the *results* of that process, per retailer.

---

## Evetech

- **Rendering:** Server-rendered (Next.js). Confirmed no headless browser
  (Playwright/Selenium) is needed — Scrapy's plain HTML fetch sees full
  page content.
- **Best data source found:** a JSON-LD `<script type="application/ld+json">`
  block, present for Google's SEO benefit, containing the *entire* product
  grid in clean structured form:
  ```
  data["@graph"][n]["mainEntity"]  ->  {"@type": "ItemList", "itemListElement": [...]}
  itemListElement[i]["item"]       ->  {"@type": "Product", "name", "url",
                                          "image", "offers": {"price",
                                          "priceCurrency", "availability"}}
  ```
  This is far more reliable than scraping visible HTML with CSS selectors,
  since retailers have their own incentive not to break SEO markup, whereas
  visual `<div>` classes get restyled/renamed casually.
- **SKU extraction:** trailing numeric ID in the product URL path
  (`.../best-deal/<id>`) — used as the per-store identifier until
  cross-store fuzzy matching exists.
- **Category tested:** GPUs (`/components/nvidia-ati-graphics-cards-21`).
  All 20 listed products captured successfully via this method.
- **Pagination:** not yet confirmed for larger categories. On the GPU page,
  `numberOfItems` in the JSON-LD matched `itemListElement` count exactly
  (20 == 20), suggesting everything fit on one page. Categories with more
  products may paginate — check for `<link rel="next">` or a `?page=`
  query param before assuming a category's full catalog fits on one crawl.
- **Verified working:** 2026-09-07 — `scrapy crawl evetech_gpu` inserted
  20 `products` and 20 `price_history` documents into MongoDB Atlas.
- **Delivery fee data:** not present in this JSON-LD block. Still an open
  item — likely needs a separate check (checkout-page or cart-API call),
  not solved yet.

---

## Wootware / Takealot / Skynamic / Rebeltech

Not yet investigated. Before writing a spider for any of these, repeat the
discovery steps below rather than guessing CSS selectors first.

---

## Discovery checklist for a new store/category

1. Fetch one real category/listing page with `scrapy shell "<url>"`.
2. Check page size: `len(response.text)`. A few KB suggests a near-empty
   JS shell (would need a headless browser); 100KB+ suggests real
   server-rendered content.
3. Search for a known product term (a model name/number you can see on the
   live site) inside `response.text` to confirm the data is actually in
   the raw HTML Scrapy receives, not just injected later by JavaScript.
4. Look for structured data *before* reaching for CSS selectors, in this
   order of preference:
   - `<script type="application/ld+json">` — schema.org structured data
     (what worked for Evetech).
   - A framework-specific embedded state blob — e.g. Next.js
     `__NEXT_DATA__`, Nuxt `__NUXT__`, or similar — search for these
     literal strings in `response.text`.
   - An actual API/XHR call the page's JavaScript makes on load (visible
     in a real browser's Network tab) that could be called directly.
5. Only fall back to CSS selectors on visible HTML if none of the above
   exist. That's the fragile option — it breaks the moment the retailer
   redesigns their page, whereas structured data rarely does.
6. Once a reliable source is found, record it here before writing the
   spider, so this file stays the shared source of truth.

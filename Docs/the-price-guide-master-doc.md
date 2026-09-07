# The Price Guide — Master Project Document

**Status:** Living document — update after every working session, then re-sync to Claude Project knowledge and the GitHub repo.
**Target:** Live and functional for Black Friday (late November), with real price history behind it.
**Owner:** Ngcebo — personal project, not coursework.

---

## How to use this document

This file has two jobs at once, on purpose:

1. It's the single source of truth for what The Price Guide *is* and how it's built.
2. It's a record of *how the project was built* — decisions, reasoning, and the prompts that led to each feature — so it can double as a practical software-engineering (SE) learning log, not just a spec sheet.

**Workflow going forward:**
- Each conversation session works on roughly one feature or decision.
- At the end of a session, update the relevant section(s) below, and add an entry to the **Decision & Prompt Log**.
- Download the updated file, replace it in your Claude Project knowledge, and commit it to the repo (e.g. `/docs/MASTER.md`) so all three stay in sync.
- Next session starts by referencing this file, so context never resets.

---

## 1. Product Vision

**Core product:** A South African PC/laptop/component price-comparison and DIY-builder web app.

Users either browse a pre-built comparison view or assemble a custom PC build, and the app shows the *true* cheapest way to buy it — factoring in that buying everything from one store vs. splitting across two or three stores changes the total once delivery fees are added.

**The honesty angle:** Beyond "cheapest price today," the app keeps historical price data so users can tell a genuine discount from a fake Black-Friday markup-then-discount. This is why price data is stored as an append-only time series from day one, not overwritten.

**Explicitly out of scope (v1):** Phones (separate future project — spec/variant complexity would pollute this schema). Cross-store cart/checkout, user accounts, and order tracking are noted as future scope, not built now.

**The bigger ambition:** If the standardized product-display approach works, it could become a reusable pattern/standard for niche DIY-tech e-commerce presentation generally (PCs today, 3D printers/SBCs later) — informative and navigable by design, instead of mimicking retailer sites that are structured to maximize time-on-site rather than help you find what you need.

---

## 2. The Business/Access Model

You don't have scraping permission or API access from most SA retailers yet, because you have no audience to offer them. The sequence:

1. Scrape conservatively and respectfully (see etiquette below) — stay under the radar.
2. Track every outbound click via your own redirect endpoint (`/out?store=X&product_id=Y`) as concrete, provable traffic data.
3. Once real numbers exist, approach non-affiliate stores with that traffic data as leverage for either an affiliate deal or explicit scraping permission — avoiding an adversarial "we caught you scraping" dynamic.

Evetech already has a public affiliate program — it's the first legitimate, sanctioned data source. Everyone else starts as a rate-limited scrape target.

**Before writing a spider for any store:** check `/docs/site-structures.md` first. It records how each retailer's data is actually structured (found via manual discovery — see Section 9.1) so that work never has to be repeated.

**Scraping etiquette (non-negotiable, baked into every scraper):**
- Respect `robots.txt`
- Rate-limit requests
- Descriptive `User-Agent` string with contact info
- Aggressive caching — don't re-fetch what hasn't changed

---

## 3. Tech Stack (decided)

| Layer | Choice | Why |
|---|---|---|
| Scraper framework | **Scrapy** | Purpose-built for structured, scheduled scraping vs. raw requests/BeautifulSoup |
| Scraper hosting/scheduling | **Zyte (Scrapy Cloud)**, free tier | Solves "run every few hours without getting blocked or babysitting a server" |
| Affiliate data source | **Evetech affiliate program** | Official feed, no scraping needed, legitimate first data source |
| Database | **MongoDB Atlas** | Document DB fits messy/inconsistent specs across stores better than rigid SQL; also a GitHub Student Pack perk |
| DB design | Two collections: `products` + `price_history` | See Section 4 |
| Backend | **FastAPI** (Flask as fallback) | REST API serving product/price data |
| Backend hosting | **Azure App Service** | $200/30-day Student Pack credit, then free tier |
| Frontend | **React** | Builder/comparison UI |
| Analytics | **SimpleAnalytics** | Privacy-friendly, 1 year free via Student Pack — pairs with click-tracker |
| Domain | **.tech via Name.com** | Free 1 year via Student Pack; leaning `rigscout.tech` |
| Dev environment | **GitHub Pro + Codespaces** | Repo hosting + cloud dev environment |

**Open / not yet decided:**
- Fuzzy-matching library for cross-store product normalization — likely `rapidfuzz`
- Whether Zyte alone handles scheduling, or Azure Functions get layered on top later

**Superseded from earlier planning (for the record — don't reintroduce):**
- ~~PostgreSQL/SQLite~~ → MongoDB Atlas
- ~~requests/BeautifulSoup/Playwright~~ → Scrapy + Zyte
- ~~DigitalOcean hosting~~ → Azure App Service

---

## 4. Data Model

**`products` collection** — static specs/metadata, one document per normalized product.
- Fields (draft): `product_id`, `canonical_name`, `category` (gpu/cpu/case/psu/etc., extensible), `specs` (flexible sub-document — this is where Mongo's schema flexibility earns its keep across wildly different categories), `store_listings` (array of `{store, store_sku, url, last_seen}`)

**`price_history` collection** — append-only time series, one document per scrape per store. Never overwritten.
- Fields (draft): `product_id`, `store`, `price`, `delivery_fee_snapshot`, `scraped_at`, `in_stock`

This split is the entire mechanism behind the Black Friday honesty feature: current price comes from the latest `price_history` entry; "is this a real discount" comes from querying the trend across the collection.

**Still to define:** exact `specs` sub-schema per category, normalization/matching key strategy (this is where `rapidfuzz` likely comes in), and how `store_listings` reconciles against `price_history` entries.

---

## 5. Standardized Product Display (extensible schema)

Goal: users shouldn't have to learn a new, deliberately-confusing navigation pattern for every retailer. One consistent way to browse/compare, regardless of category.

Approach:
- Category-agnostic core fields (name, price, price trend, store links, in-stock) render the same everywhere.
- Category-specific spec blocks (e.g. GPU: VRAM/wattage; 3D printer: build volume/nozzle type) plug into the same layout without redesigning the page.
- When a new category (3D printers, SBCs) is added, the goal is: extend the `specs` schema and a display template, not rebuild the frontend.

*(This section will get more concrete once the first category — PC components — is actually built and battle-tested. Don't over-design this before real data exists.)*

---

## 6. Data Privacy & POPIA

You are the data controller; MongoDB Atlas is a data processor — it doesn't have a contractual right to use or sell your users' data. Your real exposure is on your side of the system, not Atlas's.

**Minimums before any personal data is collected:**
- Collect the least data possible — v1 click-tracking can likely be anonymous (no accounts needed yet)
- Encrypt sensitive fields at rest
- Restrict Atlas network access to the backend's IP only
- Review and accept MongoDB's DPA from Atlas org settings
- Write a plain-language privacy notice before storing anything like an email address
- POPIA (South Africa) is the governing law here, not GDPR — revisit this section specifically once user accounts are actually being designed

---

## 7. Roadmap (Now → Next → Later)

Following SE principle: get the critical path working end-to-end before polishing anything. Frontend is *not* the critical path — data flowing into `price_history` is.

**Now (pre–Black Friday critical path):**
1. ~~MongoDB Atlas cluster + `products`/`price_history` schema live~~ ✅ Done 2026-09-07
2. ~~First Scrapy spider scraping into `price_history` on a schedule via Zyte~~ ⚠️ Partially done 2026-09-07 — spider runs manually and successfully (Evetech GPUs, 20/20 products), but not yet scheduled via Zyte. Manual `scrapy crawl` only so far.
3. Evetech affiliate feed ingested as second data source
4. Click-tracker redirect endpoint (`/out?store=X&product_id=Y`) logging clicks

**Next:**
5. FastAPI endpoints serving product + price-trend data
6. Basic React comparison view (read-only, no builder yet)
7. Fuzzy-matching pass for cross-store product normalization

**Later:**
8. DIY builder + combinatorial optimizer (single-store vs. split-purchase)
9. Standardized display template extended to a second category (proves the schema generalizes)
10. Approach additional retailers using traffic data as leverage

**Future / explicitly out of scope for now:**
- User accounts, cross-store cart, order tracking
- Phone specification database (separate project)

---

## 8. Decision & Prompt Log

*Add an entry per session. This is the "why" trail — what was decided, and what prompt/conversation led to it.*

| Date | Session topic | Decision made | Rationale / prompt origin |
|---|---|---|---|
| (Phase 0) | Initial scoping | Full stack chosen: Scrapy/Zyte, MongoDB Atlas, FastAPI, Azure App Service, React, SimpleAnalytics, .tech domain | Original brainchild conversation — see archived transcript |
| 2026-09-03 | Master doc creation | Consolidated Phase 0 stack with new additions: prompt log, POPIA section, standardized display schema, learning-log framing | This conversation — reconciling drift between an earlier (incorrect) Postgres/BeautifulSoup assumption and the actual decided stack |
| 2026-09-07 | First working scraper + first real price_history data | Built `price_guide_scraper` (Scrapy project, MongoDB pipeline, Evetech GPU spider). Fixed venv/dotenv setup, diagnosed Atlas connection timeout as school-wifi port 27017 blocking, discovered Evetech embeds full product data as JSON-LD structured data (built for Google SEO) rather than requiring fragile CSS scraping. Confirmed 20/20 products + price_history rows inserted successfully. Rotated an accidentally-exposed DB password. | Session started as "get the background scraping infrastructure running before Black Friday." See `/docs/site-structures.md` for the JSON-LD discovery details, kept separately so future stores/categories can reuse the approach. |

---

## 9. Documentation-as-Learning Practice

The point of this project, beyond the product itself, is to actually *apply* SE principles while building with AI assistance — not to hand it off to an agent and end up with a finished product you can't explain or maintain.

Practices being deliberately followed:
- **Iterative delivery** — critical path first (Section 7), features fleshed out incrementally rather than built all at once
- **Decision logging** — Section 8, so every choice has a traceable rationale
- **Separation of concerns** — data collection, storage, API, and UI are distinct layers that can be worked on independently
- **Documentation as a first-class deliverable**, not an afterthought — this file itself
- No dedicated tester/feedback loop exists yet (hardware-enthusiast niche, no user base) — noted as an open risk, not solved yet. Worth revisiting once there's *any* real traffic.

### 9.1 Why we inspect before we scrape

A scraper has no eyes. It doesn't see a webpage the way a person does — no
visual grid of product cards, no colours, no obvious "this is the price"
label. All it gets is one long string of raw text: HTML tags and
sometimes JSON, with nothing marking where one product ends and the next
begins. There's no way to know in advance how a given retailer has
arranged that text — every site does it differently, and it changes
without notice.

So before a single line of a spider gets written, we have to feel our way
through that raw text by hand — a few lines at a time in `scrapy shell`,
poking at it: "does this piece of text exist? does this tag repeat once
per product? is the real data actually hidden somewhere less obvious,
like a block of JSON stuffed into a `<script>` tag for search engines to
read?" That poking-around process isn't wasted time or a sign of not
knowing what we're doing going in — it *is* the work. There's no shortcut
to it, because the layout is unknown until we look.

What we're hoping to find each time is a **repeatable shape**: some
structure that shows up consistently for every product, that we can point
code at reliably. Sometimes that shape is obvious HTML. Often, as with
Evetech, it turns out retailers already publish a clean, structured
version of their product data for Google's benefit (JSON-LD) — and reading
that is far more reliable than guessing at CSS class names a designer
might casually rename tomorrow.

Because this discovery work is expensive to redo and easy to lose, the
findings for each retailer get written down in `/docs/site-structures.md`
rather than staying in one person's memory (or one AI conversation).
Before writing a spider for any new store or category, check that file
first — the discovery may already be done. If it isn't, that file also
lists the order of things worth checking (JSON-LD first, then other
embedded data blobs, CSS selectors only as a last resort), so the search
itself doesn't have to be reinvented either.

---

## 10. Open Questions

- Fuzzy-matching library choice (leaning `rapidfuzz`)
- Zyte-only scheduling vs. layering Azure Functions later
- Exact `specs` sub-schema per product category
- How/when to find early testers given the niche, no-audience starting point

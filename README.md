# PC-PRICE-GUIDE

A South African PC, laptop, and component price-comparison and DIY-builder web app.

## What it is

The Price Guide lets you either browse a pre-built comparison view or assemble a custom PC build, and shows you the **true cheapest way to buy it** — not just the lowest sticker price, but the lowest *total* once delivery costs are factored in across stores. Buying every part from one retailer isn't always cheaper than splitting the order across two or three once shipping is added; this app does that math for you.

It pulls live prices from South African PC retailers (Wootware, Evetech, Takealot, and others), keeps a running history of every price change, and helps you tell a genuine deal from a markup-then-discount trick — a pattern common enough around Black Friday that it deserves its own feature, not an afterthought.

## Why it exists

Two problems, one project:

1. **Price comparison across SA retailers is manual and tedious.** There's no single place to check whether Wootware, Evetech, or Takealot has the better deal on a specific part today, let alone the better deal once delivery is added.
2. **"Discounts" aren't always real.** Retailers can inflate a price before a sale event and then "discount" back to a normal price. Without price history, there's no way to tell the difference. This app stores every scraped price as an append-only historical record from day one, so trends are verifiable, not just claimed.

The longer-term goal: if the standardized, no-nonsense way of presenting products here works well for PCs, the same pattern extends to other DIY/niche tech categories (3D printers, single-board computers) later — one consistent, informative browsing experience instead of retailer sites designed to maximize time-on-site rather than help you find what you need.

## Core features

- **Cross-store price comparison** for equivalent products, normalized across retailers
- **True-cost calculation** — per-store delivery fees factored into the final price, including split-purchase comparisons
- **Historical price tracking** — append-only time series per product per store, powering real discount verification
- **DIY builder** with a combinatorial optimizer (single store vs. split purchase) — *planned, not yet built*
- **Direct outbound links** to retailers to complete purchases (no cross-store cart/checkout — see Scope)

## Scope

**In scope (v1):** PC components, laptops, and prebuilt PCs from South African retailers.

**Explicitly out of scope (for now):**
- Phones (spec/variant complexity warrants a separate project)
- Cross-store cart, checkout, or order tracking
- User accounts (v1 works without them)

## Tech stack

| Layer | Choice |
|---|---|
| Scrapers | Python + Scrapy |
| Scraper scheduling/hosting | Zyte (Scrapy Cloud) |
| Database | MongoDB Atlas (`products` + `price_history` collections) |
| Backend | FastAPI |
| Backend hosting | Azure App Service |
| Frontend | React |
| Analytics | SimpleAnalytics |

Full architecture, data model, and decision history are documented in [`/docs/MASTER.md`](./docs/MASTER.md).

## Status

Actively in development, targeting a working v1 (live price comparison + real price history) for Black Friday. This is a personal project, built openly and documented step by step as both a product and a software-engineering learning exercise — see `/docs/MASTER.md` for the full decision log and reasoning behind each feature.

## Data & privacy

South African users' data is handled under POPIA. MongoDB Atlas is used strictly as a data processor — it does not have rights to use or sell data stored here. No personal data is collected beyond what's strictly necessary (e.g. v1 click-tracking is anonymous). Details in `/docs/MASTER.md`.

## License

TBD.

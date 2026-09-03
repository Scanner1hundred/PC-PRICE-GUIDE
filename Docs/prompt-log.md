# The Price Guide — Prompt & Decision Log

**Purpose:** This is the "why" trail — the actual prompts/conversation moments that shaped what the app *is* and *does*. Tech stack tooling choices (which database, which host, etc.) are documented separately in `tech-stack.md`. This file is about product scope, features, and business logic only.

---

## 1. The core concept

> "can we create a webapp that searches all of these websites for laptops, pc components and pre-built prices and specifications so basically it's a DIY PC builder or a prebuilt comparing site and then compared the price per component and share the link to the cheapest or the best deal amongst all the stores. taking delivery into consideration as separate delivery of each component might be more expensive than buying multiple components in the same store."

**What this established:** the core product — a cross-store PC/laptop/component comparison and DIY-builder tool for South African retailers, with delivery-cost-aware price optimization as a defining feature (not just "cheapest listed price," but cheapest *total* once delivery is factored in, including the single-store-vs-split-purchase tradeoff).

---

## 2. Wanting it fully live, not a mockup

> "how can we make live one with active web scrape. with a dedicated backend. and everything working as required"

**What this established:** the requirement for a real, live, automatically-updating system — actual scrapers, a real database, a real backend — rather than a static demo or artifact-only prototype. This is the prompt that moved the project from "concept" to "real infrastructure."

---

## 3. The idea of asking for permission instead of just scraping

> "to not be blocked from scraping can't I email the sites for access to scrap their sites as it might bring traffic to their sites?"

**What this established:** the seed of the business/access model — the idea that traffic/attention could be offered to retailers as leverage for legitimate access, rather than treating scraping purely adversarially.

---

## 4. The full business model, click tracking, and phone scope decision

> "so for now let's limit scraping to prevent IP blocks and also let's look for websites that have affiliate programs even if they don't we should scrape them too just limiting that. once we have an audience we can then email them with evidence that our website is driving traffic to their site and express interest in either having an affiliate program or unlimited scraping access and no IP block also we need to track link clicks as we can't realistically track how much time each person spends at a particular website. at a later stage we can add phones but for now it's PCs, laptops and computer components. we might need a dedicated one for phones since that database would be a crime to society."

**What this established (this is the single most consequential prompt in the project so far):**
- The full sequencing strategy: scrape conservatively now → build an audience → use traffic evidence to negotiate affiliate deals or scraping permission later
- Click tracking as the core evidence-gathering mechanism (this became the `/out?store=X&product_id=Y` redirect endpoint)
- Explicit scope decision: PCs, laptops, and components only for v1
- Explicit scope exclusion: phones deferred to a separate future project, due to spec/variant complexity

---

## 5. Deadline pressure and the historical pricing feature

> "i want the webapp to be deployed by November for black friday. Plus i dont know if i will be able to afford SimpleAnalytics after the free month by then i dont think i would have generated enough traffic... now i am already thinking about an additional feature like historical pricing to see if black friday prices are deals or scams... this database need to be robust for sure so you can imagine my concern with MongoDB acting up"

**What this established:**
- The hard deadline: live and functional for Black Friday (late November)
- The historical price-tracking / "honesty" feature — letting users tell a genuine Black Friday discount from a fake markup-then-discount. This directly shaped the database design decision to store price data as an append-only time series (`price_history` collection) rather than overwriting current prices, from day one rather than retrofitted later.
- Implicit requirement that the database needs to be reliable/robust given how central it now is to a user-facing trust feature, not just a data store.

---

## 6. Branding direction

> "what would be the best url name .tech for the web app?"

**What this established:** the domain/branding exploration — several options were suggested (`rigscout.tech`, `pcscout.tech`, `buildcompare.tech`, etc.), but none were used. The domain actually registered was **`pcpriceguide.tech`** — a self-chosen, descriptive name that directly states what the app does, in keeping with the project's own name ("The Price Guide").

---

*Add new entries here as future sessions introduce new product decisions, features, or scope changes. Keep tooling/infrastructure decisions in `tech-stack.md` instead.*

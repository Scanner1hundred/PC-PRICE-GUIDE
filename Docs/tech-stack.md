# The Price Guide — Tech Stack Documentation

**Status:** Living document — update whenever a stack decision changes.
**Companion file:** see `prompt-log.md` for the decision/prompt trail behind *why* the app exists and what it does — this file is purely the *what we're building with*.

---

## Decided Stack

| Layer | Choice | Why |
|---|---|---|
| Scraper framework | **Scrapy** (Python) | Purpose-built for structured, scheduled scraping vs. raw requests/BeautifulSoup |
| Scraper hosting/scheduling | **Zyte (Scrapy Cloud)**, free tier | Solves "run every few hours without getting blocked or babysitting a server" |
| Affiliate data source | **Evetech affiliate program** | Official feed, no scraping needed, legitimate first data source |
| Database | **MongoDB Atlas** | Document DB fits messy/inconsistent specs across stores better than rigid SQL; also a GitHub Student Pack perk |
| DB design | Two collections: `products` + `price_history` | Append-only time series powers the Black Friday honesty/history feature |
| Backend | **FastAPI** (Flask as fallback) | REST API serving product/price data + click-tracking redirect endpoint |
| Backend hosting | **Azure App Service** | $200/30-day credit via the standard Azure Free Account (not the student tier — see note below), then Azure's always-free tier services for 12 months |
| Frontend | **React** | Builder/comparison UI |
| Analytics | **SimpleAnalytics** | Privacy-friendly, 1 year free via GitHub Student Pack — pairs with click-tracker |
| Domain | **pcpriceguide.tech** via Name.com | Free 1 year via GitHub Student Pack; self-chosen, descriptive name |
| Dev environment | **GitHub Pro + Codespaces** | Repo hosting + cloud dev environment |

**Open / not yet decided:**
- Fuzzy-matching library for cross-store product normalization — likely `rapidfuzz`
- Whether Zyte alone handles scheduling, or Azure Functions get layered on top later

**Superseded from earlier planning (for the record — don't reintroduce):**
- ~~PostgreSQL/SQLite~~ → MongoDB Atlas
- ~~requests/BeautifulSoup/Playwright~~ → Scrapy + Zyte
- ~~DigitalOcean hosting~~ → Azure App Service
- ~~Heroku~~ → Azure App Service (standard Free Account, after Azure for Students verification failed)

---

## Platform Notes & Gotchas

### Azure — student verification vs. standard free account
GitHub Student Developer Pack approval and **Azure for Students** approval are handled by two completely separate verification systems. Being accepted into the GitHub Pack does **not** guarantee Azure for Students eligibility — Azure runs its own academic verification independently, and it can reject you even when GitHub accepted you. This is a common, well-documented mismatch, not a sign anything was done wrong on setup.

**Why Azure for Students specifically can reject an eligible-seeming applicant:**
- Requires age 18+ (Azure for Students specifically — there's a separate, more limited **Azure for Students Starter** offer for under-18s, which doesn't include the same $100 credit or full service access)
- Requires enrollment at an accredited, degree-granting 2- or 4-year institution — MOOCs, bootcamps, and for-profit training providers are explicitly excluded
- Requires verification via an institutional (school-issued) email address the automated system recognizes — some legitimate institutions simply aren't in Microsoft's recognized-domain database, causing an automatic fail even with a valid academic email
- One Azure for Students subscription per person, ever — a prior redemption on any account blocks a new one

**This project's path:** Azure for Students verification didn't go through, so the project is running on the **standard Azure Free Account** instead — $200 credit for the first 30 days, then Azure's always-free tier (65+ services) plus 12 months of free monthly amounts on 20+ popular services. Requires a card on file for identity verification, but spending protection means no auto-charge unless manually upgraded to pay-as-you-go. This is a fine substitute for this project's needs and timeline.

### MongoDB Atlas — GitHub sign-in
Atlas's "Sign in with GitHub" requires a **public** (not just verified) email on the GitHub account, and even then the in-app "link GitHub" option isn't guaranteed to be visible/available depending on account state. Workaround used here: sign up for Atlas directly with email/password, redeem the Student Pack activation code against that account (redemption isn't tied to login method), and skip GitHub linking entirely — it isn't required for anything downstream.

---

## Deployment Split (for clarity)
- **Zyte** runs and schedules the scrapers independently — writes directly to MongoDB Atlas.
- **Azure App Service** hosts the FastAPI backend (product/price API + click-tracking redirect endpoint) independently — reads/writes to the same MongoDB Atlas cluster.
- The two services never talk to each other directly. MongoDB Atlas is the single shared point of contact between data collection and the API layer. This keeps scraping load from ever competing with API/backend compute.

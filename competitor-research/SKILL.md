---
name: competitor-research
description: Structured competitive research on any B2C, B2B, B2B2C or D2C company: Snapshot, Deep Dive or Battle Card tiers, optional vs-baseline, every fact sourced. Use for "research/compare/look into X".
---

# Competitor Research

**Triggers.** Any request to research, profile, size up, tear down, benchmark or compare a company, competitor, startup or product; questions like "who are X's competitors", "how does X make money", "what does X charge", "build a battle card for X", "how do we stack up against X"; or a request to refresh an existing competitor doc. A plain "look into <company>" is enough.

Treat every request as a **run** of a small product, not a one-off search. A run has a config (what to research, how deep, against whom, rendered how), a fixed pipeline, a fixed data schema, and a stable output layout. Stability is the point: two runs a month apart on the same company must be directly comparable, and a run on a D2C brand must look like a run on a B2B2C platform so the user never has to re-learn the format.

## 1. Run config (the "form")

Fill this before any search. Infer what you can from the request; show the resolved config in one line ("Run: Snapshot · Razorpay · vs Stripe · Doc") and start. Ask only when a field is both unknown *and* changes the result materially — target ambiguity (two companies share the name), or Battle Card with no baseline.

| Field | Values | Default |
|---|---|---|
| `target` | company name or URL | required |
| `mode` | `snapshot` · `deep-dive` · `battle-card` | `snapshot` |
| `baseline` | `none` · `<any company or product>` | `none`; Battle Card requires one |
| `output` | `doc` · `dashboard` · `chat` · `xlsx` | by mode (below) |
| `segment` | `b2c` · `b2b` · `b2b2c` · `d2c` · `auto` | `auto` — resolved in step 2 |
| `region` | market focus, e.g. `IN`, `US`, `global` | infer from target; else `global` |

Default output by mode: Snapshot → `chat`; Deep Dive → `doc`; Battle Card → `doc`. `xlsx` is an add-on the user can request alongside any mode. `dashboard` is only worth it with two or more targets (a landscape view); for a single company steer to `doc` and say why in one line.

When the baseline is the user's own company or product, its public site is usually thinner than what the user knows. Ask for the internal facts the comparison needs (actual pricing, feature status, named customers) rather than researching them from outside, and tag them `internal` in sources. If the user declines or does not answer, research the baseline from public sources like any other company and say so in the Run summary and the Comparison section, so nobody mistakes list price for the price actually quoted. If the baseline is a multi-product company, pick the product line closest to the target's segment and name the choice in the run line so the user can override.

### Tier budgets

Budgets exist so a casual lookup stays cheap and a teardown stays thorough. Stop collecting when the budget or the schema is exhausted, whichever comes first; never pad.

| Mode | Searches | Pages fetched | Browser (gated) | Modules filled | Target wall time |
|---|---|---|---|---|---|
| Snapshot | ≤ 10 | ≤ 8 | none | Identity, Product, Pricing, Traction (headline only), Recent moves | ~5 min |
| Deep Dive | ≤ 40 | ≤ 30 | up to 6 pages | all modules | ~20 min |
| Battle Card | ≤ 25 | ≤ 20 | up to 4 pages | Product, Pricing, Customers & reviews, Traction (headline only), Recent moves, Comparison, plus battle-card blocks | ~12 min |

Counting rules: a fetch that redirects or returns an empty page still counts (it cost time, and the footer should show that a page was attempted). Search results often return titles and URLs without snippets, so fetches, not searches, are usually the binding constraint; spend the first fetch on the page most likely to answer several fields at once (pricing, then about, then a recent funding article). Facts picked up incidentally for a module outside the tier's list are kept and shown under that module with the prefix "Incidental:", never dropped.

## 2. Pipeline

Run the stages in order. Each stage has an exit condition; do not move on without meeting it.

**Stage 0 — Prior state.** If `output` is `doc`, check Claude Docs for an existing doc titled `Competitor: <target>` (if the doc surface is a file instead, look for `Competitor - <target>.md` in the working folder). If one exists, you are doing a *refresh*: read it, keep its sections, and at the end write a "What changed since <last run date>" block at the top. Refreshes are the main reason this skill exists; a user tracking five competitors wants deltas, not five fresh essays.

**Stage 1 — Resolve the entity.** Establish official domain, legal/brand name, HQ, founding year, and whether it is independent or a subsidiary. Exit: you can name the domain and one disambiguating fact. If two plausible companies match, stop and ask with a one-line description of each.

**Stage 2 — Classify the segment.** Decide B2C / B2B2C / D2C from who pays and who uses, not from how the company describes itself:

- **B2C** — consumer pays, consumer uses (apps, marketplaces, subscriptions). Evidence lives in app stores, Trustpilot/Reddit, traffic estimates.
- **B2B** — a business pays, its own staff use (per-seat SaaS, workspace tools, dev tools). Evidence lives in G2/Capterra, pricing page, docs, careers page, LinkedIn. Metrics that matter: seat price, free-to-paid motion, enterprise gating.
- **B2B2C** — a business pays, but *its* customers, members or employees are the end users and often receive value through the platform (rewards and incentives, embedded finance, loyalty engines, white-label anything). Same evidence as B2B plus partner pages and API docs. Metrics that matter: take rate, float, breakage, end-user reach.
- **D2C** — a brand sells its own physical product online, mostly to consumers. Evidence lives in the storefront, marketplaces (Amazon/Flipkart/Nykaa), Instagram, funding press, reviews.

Many companies are mixed (a D2C brand with a B2B gifting arm; a B2B tool with a large free prosumer base). Pick the primary, note the secondary. The segment changes which metrics matter and where you look, so getting it wrong wastes the whole budget.

**Stage 3 — Collect, module by module.** Follow the source order in section 3. Work the official site first (it is authoritative on claims and pricing), third-party next (it is authoritative on reality), news last (it is authoritative on dates). Record every fact into the schema in section 4 with its source URL, publication/observation date, and a confidence tag. Leave a field as `Not found` rather than reasoning your way to a number — a fabricated headcount in a doc that reaches a leadership meeting is the worst outcome this skill can produce.

**Stage 4 — Browser escalation (Deep Dive and Battle Card only).** When search leaves a high-value field empty — headcount and growth, G2 rating and review themes, funding rounds, traffic band, app ratings — open the gated source in the user's browser. Read-only: never log in, never follow, connect, message, export or change anything on an account. Record that the figure came from a logged-in view (it matters for attribution if the doc is shared outside the company). If a site asks for approval and it is declined, mark the field `Not found (gated)` and move on.

**Stage 5 — Synthesize.** Only now write the judgement parts: positioning read, strengths and weaknesses, threat level, "so what for us". Every judgement must point back to at least one schema fact; if you cannot cite the fact, cut the judgement.

**Stage 6 — Quality gate.** Before rendering, check:

- Entity confirmed and segment stated with reasoning.
- No number without a source and a date. Pricing has an "as seen on" date.
- Company-claimed vs third-party figures are visibly distinguished (e.g. "claims 2,000 customers (site)" vs "~450 employees (LinkedIn, Oct 2026)").
- Contradictions between sources are shown, not silently resolved.
- Estimates (traffic, revenue, headcount from directories) are given as bands, not point values.
- Comparison section, if present, compares like with like (same plan tier, same region, same currency).

**Stage 7 — Render** using section 5, then append the run footer.

## 3. Modules and source playbook

Each module lists what to capture and where to look, in order. Segment notes say what to weight.

**Identity & positioning** — tagline, stated ICP, category they claim, category the market puts them in, HQ, founded, ownership. Sources: homepage, /about, LinkedIn "About", Crunchbase. All segments.

**Product & features** — core offering, feature list grouped by job-to-be-done, integrations/API surface, platforms (web/iOS/Android), what is notably *missing*. Sources: product pages, docs, changelog/release notes, app store listing. B2B2C: weight integrations, API docs, admin controls, white-labelling, compliance pages. B2C: weight app-store "What's new" cadence and ratings. D2C: weight SKU range, hero products, bundle structure.

**Pricing & monetization** — pricing model (per seat / usage / take rate / subscription / one-time), published tiers with amounts and currency, free tier or trial, enterprise gating, discount behaviour. Sources: /pricing, app-store IAP list, storefront, Wayback Machine for pricing history, review sites for "pricing" complaints. D2C: price bands, AOV signals, discount depth (coupon sites, sale frequency). B2B2C: who pays whom — if the end user is paid *through* the platform (rewards, payouts), capture take rate, float, breakage if disclosed.

**GTM & distribution** — primary channels, sales motion (PLG / inside / field / partner), marketplace presence, geographic footprint, notable partnerships, content/SEO strength. Sources: careers page (sales roles reveal motion), partner directory, marketplace listings, LinkedIn posts, ad libraries. D2C: own-site vs marketplace split, quick-commerce presence, offline retail.

**Traction** — funding (round, amount, date, lead investors), headcount and 6/12-month growth, revenue if disclosed or credibly reported, customer count claims, app downloads and ratings, web traffic band, geographic mix. Sources: Crunchbase/Tracxn/press for funding; LinkedIn for headcount; data.ai / Sensor Tower-cited press for downloads; Similarweb for traffic; MCA filings or Tofler-cited press for Indian revenue. Label every one of these as estimate or disclosed.

**Customers & reviews** — named customers and logos, segment mix, review score and count per platform, top three praise themes, top three complaint themes with representative quotes (short, attributed). Sources: G2, Capterra, Trustpilot, app stores, Reddit, Glassdoor (culture signal only, not product). This module carries most of the "why they lose" material, so quote complaints precisely.

**Team & hiring** — founders and key execs with prior companies, leadership changes in 12 months, open roles by function (a hiring spike in a function is a roadmap leak). Sources: LinkedIn, careers page, press.

**Recent moves** — last 12 months: launches, pricing changes, funding, layoffs, acquisitions, major partnerships, regulatory events, outages. One line each with date. One older anchoring event (an acquisition or relaunch that explains everything since) may be included, marked "outside window". Sources: news search, company blog, LinkedIn posts, release notes.

**Comparison** (when `baseline` set) — side-by-side on positioning, ICP, pricing model and representative price, feature parity matrix (✓ / partial / ✗ with a note), traction headline, named shared customers or lost deals if public. Pull the baseline's facts with the same rules; when the baseline is the user's own product, use the internal facts they supplied over the public site.

**Assessment** — strengths, weaknesses, threat level to the baseline (Low / Medium / High with the single biggest reason), three things to watch, and a "so what for us" paragraph that names a decision the reader could make this quarter.

## 4. Canonical schema

Keep this shape internally for every run, whichever output is chosen. Renderers read from it; nothing is written straight into a doc or sheet without passing through here.

```yaml
run: {target, mode, baseline, segment, region, date, searches_used, pages_fetched, browser_pages, gaps: []}
identity: {domain, legal_name, hq, founded, ownership, tagline, stated_icp, category_claimed, category_market}
product: {core_offering, features: [{group, item, note}], integrations: [], platforms: [], notable_gaps: []}
pricing: {model, currency, tiers: [{name, price, unit, includes}], free_tier, enterprise_gated, discount_notes, as_seen_on}
gtm: {channels: [], sales_motion, marketplaces: [], geographies: [], partnerships: []}
traction: {funding: [{round, amount, date, lead}], headcount: {value, growth_6m, growth_12m}, revenue, customers_claimed, downloads, ratings: [{store, score, count}], traffic_band}
customers: {named: [], segment_mix, reviews: [{platform, score, count}], praise_themes: [], complaint_themes: [{theme, quote, source}]}
team: {founders: [], execs: [], changes_12m: [], open_roles_by_function: {}}
recent_moves: [{date, event, source}]
comparison: {positioning, icp, pricing, feature_matrix: [{feature, target, baseline, note}], traction, shared_customers}
assessment: {strengths: [], weaknesses: [], threat_level, threat_reason, watch: [], so_what}
sources: [{id, url, title, date, type: official|third_party|news|gated, used_for: []}]
```

Every scalar that came from a source carries `(source_id, date, confidence)` where confidence is `disclosed` (company said it), `reported` (credible third party), `estimate` (directory or model-derived), `calc` (derived by you from sourced inputs, e.g. a break-even between two pricing units — show the inputs), or `internal` (supplied by the user). Render it inline as `(S3 · Oct-26 · reported)` after the value, with `S3` pointing at the numbered Sources list; this works in chat, Markdown tables, docs and sheet cells alike.

## 5. Renderers

Same schema, four surfaces. Section order never changes between runs or companies; a module with nothing in it still appears with a one-line "Not researched in Snapshot" or "Not found" so the reader knows it was looked for.

**`chat`** — Lead with a 5-line header: one-sentence what they do, segment, pricing model with one representative price, traction headline, threat level (write "n/a — no baseline" when none is set, so the layout stays identical across runs). Then one compact table per filled module, then Assessment as short prose, then Sources as a numbered list. No headers deeper than one level; the user reads this on a phone.

**`doc`** — Title `Competitor: <target>` (append ` vs <baseline>` when set). Create the skeleton first with these headings in this order, then fill: Run summary · What changed (refresh only) · Identity & positioning · Product & features · Pricing & monetization · GTM & distribution · Traction · Customers & reviews · Team & hiring · Recent moves · Comparison (if baseline) · Assessment · Battle card (Battle Card mode only) · Sources · Run footer. Use tables for tiers, feature matrix, reviews and funding. Dates and people as chips where the doc supports it. On a refresh, update sections in place and move superseded figures into the "What changed" block with old → new and the date.

**`dashboard`** — Only for a landscape of 2+ targets. Load the `dataviz` skill before building. Tiles: one per target with segment, funding, headcount band, review score, threat level. Below: a feature parity matrix, a pricing comparison table, and a recent-moves timeline. Every number links to its source. Persist with the Artifact tool so the user can reopen it; name it `Competitive landscape: <space>`.

**`xlsx`** — Load the `xlsx` skill. One workbook, sheets: `Profile` (key–value), `Features`, `Pricing`, `Funding`, `Reviews`, `Recent_Moves`, `Comparison` (if baseline), `Sources`. First row headers, frozen; a `source_id` column on every fact row; a `confidence` column on every numeric row. Name it `<YYYY-MM-DD>_<target>_competitor_research.xlsx`.

### Battle Card block (Battle Card mode)

Written for a salesperson who has ninety seconds before a call. Append after Assessment:

- **One-liner on them** (what they'd say about themselves) and **one-liner for us** (how to reframe).
- **Why we win** — three to five points, each tied to a schema fact and ideally a customer complaint theme about them.
- **Why we lose** — their strengths and the baseline's own complaint themes from reviews; a card that hides these gets the rep ambushed.
- **Landmines** — questions to plant that expose their weaknesses.
- **Objection handling** — their likely claims about us, with a response each.
- **Pricing in the room** — their representative price next to ours, same unit and currency, with the date.
- **Proof points** — shared or switched customers, public numbers.

### Run footer (every output)

`Run: <mode> · <date> · <searches_used> searches · <pages_fetched> pages · <browser_pages> browser pages · Gaps: <comma list or "none"> · Confidence: <High/Medium/Low — one reason>`. Gaps are fields that stayed `Not found`; listing them tells the user exactly what a Deep Dive would add.

## 6. Judgement calls

- A thin public footprint is itself a finding (early-stage, stealth, or B2B2C with no self-serve). Say so instead of stretching weak sources.
- Prefer the most recent source when two disagree, but show both when the gap matters (headcount 300 vs 800 is a story, not noise).
- Indian targets: check MCA/Tofler-cited revenue, Inc42/Entrackr/YourStory for funding, Play Store over App Store for ratings.
- If the user supplies internal knowledge (lost deals, pricing seen in RFPs), tag it `internal` in sources and never render it into a dashboard that might be shared.
- When the target turns out to be a partner or customer rather than a competitor, say so up front; the research still runs, but the Assessment should talk about dependency and overlap, not threat.
- Push back once if the user asks for Battle Card without a baseline, or Deep Dive on a company whose entire footprint is a landing page — then do what they ask.

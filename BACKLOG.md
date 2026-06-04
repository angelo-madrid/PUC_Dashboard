# PUC Dashboard — Backlog

**Version:** 1.0.1
**Last updated:** 2026-06-04 16:00 PHT
**Maintainer:** Angelo Madrid

Pending work, open questions, and decisions to make. Items grouped by priority and theme.

> **Versioning convention:** Semantic versioning. **Major** (X.0.0) for major reorganization of the backlog itself. **Minor** (1.X.0) when significant new themes or workstreams are added. **Patch** (1.0.X) for adding/closing individual items. Bump the version, update the timestamp, and add a changelog entry whenever you edit this file.

---

## 🔥 Active — needs decision/action this week

### Visual review of the scaffolded landing page

Before pasting the Taytay dashboard into `stores/taytay/index.html`, walk through the live site at https://angelo-madrid.github.io/PUC_Dashboard/ and answer:

- [ ] Does the landing page aesthetic match the Taytay dashboard styling? (DM Serif Display + DM Sans, cream background, ink text, terracotta accent.)
- [ ] Do the 10 store cards read well? Does Taytay's "LIVE" status stand out clearly from the 9 greyed "Planned" cards?
- [ ] Does the masthead copy land right ("PUC Dashboard — Nutrizone Food Corp · SP Madrid")?
- [ ] Click into `stores/taytay/`. Is the placeholder note clear enough that you (or Vincent, if he stumbles on it) won't be confused?

**Outcome:** any required CSS/copy tweaks to feed into the next Claude Code prompt.

### Capital call decisions (May 31, 2026 deadline)

- [ ] Confirm CAPEX basis: is the PUC tracker's ₱2.2M per-cart figure equipment-only, or does it include franchise fee + WC? Affects total capital call amount (₱14.1M full vs ~₱12.4M equipment-only).
- [ ] Site 4 (Amang Rodriguez): lease starts July 1, after June 13 target opening. Hold capital release or proceed?
- [ ] Site 7 (PLDT): award notice "Not Started" but June 8 target shown. Reclassify or push date?
- [ ] Eton Centris: confirm whether ₱4M total outlay assumption holds (₱3.5M tracker CAPEX + ~₱500K WC), or get its actual budget breakdown.

---

## 📊 Data architecture (next Claude Code session)

### Separate operating from financial data

Build a `/data/` library with clear ownership and cadence:

```
data/
├── operating/
│   ├── taytay_daily.json          ← Vincent updates weekly from EOD reports
│   └── _schema_daily.json         ← Field definitions, validation
├── financial/
│   ├── taytay_pnl.json            ← Finance updates monthly
│   ├── taytay_debt_schedule.json  ← Static, set at loan origination
│   ├── taytay_budget.json         ← Updated start of year
│   └── _schema_pnl.json
├── shared/
│   ├── cost_assumptions.json      ← Variable cost % and fixed opex
│   └── seasonality_factors.json   ← Rain + holiday multipliers
└── README.md                      ← How to update each file
```

**Why:** different update cadences, different owners, different validation needs. Pulling shared model parameters into one place means recalibration touches one file.

### Action items

- [x] **Decide JSON schema conventions** (date format, naming, optional fields) — locked in `DATA_ARCHITECTURE.md` v1.0.0
- [x] **Design the data folder structure** — store-first layout, monthly operating files, locked in `DATA_ARCHITECTURE.md` v1.0.0
- [ ] Write a `data/README.md` explaining how Vincent updates the operating file weekly
- [ ] Refactor Taytay dashboard to `fetch()` from JSON files instead of embedded `DATA` const
- [ ] Add a validation pass (or just a smoke test) so a typo in JSON doesn't silently break the dashboard
- [ ] Decide if Vincent edits via GitHub web UI (foolproof structure needed) or someone else does it for him

---

## 🏗️ Model & analysis backlog

### Eton Centris in-line economics

The model treats it as a ₱4M cart equivalent, but it's a different format with:
- Higher rent (in-line vs forecourt)
- More staff (seating means service)
- Higher ticket size (sit-down vs gas-and-go)
- Likely different daily run-rate profile

- [ ] Get the actual Eton Centris cost breakdown (similar to the Taytay screenshot)
- [ ] Build a separate unit-economics model for in-line format
- [ ] Stress-test fleet model with Eton at lower margin than carts

### Stress-case toggle on dashboard

- [ ] Add UI toggle: Base / Conservative / Stress scenarios
- [ ] Let users see the model flex live during partner reviews

### Seasonality recalibration

- [ ] Track June 2026 daily actuals and compare to the −15% modeled factor
- [ ] By end of August (peak monsoon), have enough data to calibrate real factors vs the proxy
- [ ] Consider adding weather-correlation tracking in EOD reports (Vincent already notes "bad weather" — formalize it)

### Higher-ceiling sensitivity case

- [ ] Late-May Taytay run-rate was ₱31,468/day (last 7d). If sustainable, mature run-rate may be higher than the modeled ₱28K. Add an "upside" scenario where mature = ₱31-32K.

### Quarterly review template

- [ ] Build a partner-facing quarterly review doc: variance analysis, debt service status, equity trajectory, decision points for Store 11+

---

## ⚙️ Operations & process

### Weekly update workflow

- [ ] Define who updates the operating JSON each week. Vincent? An ops admin? Angelo?
- [ ] If Vincent, walkthrough document showing exactly how to edit via GitHub web UI
- [ ] Set a standing day/time (e.g., "Monday 10am: prior week's data goes in")
- [ ] Validation: dashboard should flag if data is more than 10 days stale

### Monthly financial close

- [ ] Define who updates the financial JSON each month
- [ ] Tie to BIR Sales Summary export (currently the canonical monthly source)
- [ ] Reconciliation step: ensure daily sum = monthly total

### Permissions & sensitivity

- [ ] The repo is public — fine for Pages, but consider whether store sales numbers should be in a public repo
- [ ] Options: (a) keep public, accept transparency; (b) make repo private + use deploy workflow that pushes built site to a public Pages branch; (c) redact specific numbers in the public version, keep raw in private

### Mosaic POS automation

- [ ] Investigate whether Mosaic POS has an API or scheduled export capability
- [ ] If yes, scope a small Python script (like Investment_Dashboard's `generate_dashboard.py`) to pull daily and update the JSON
- [ ] Long-term goal: same automation pattern as Investment_Dashboard (GitHub Actions + scheduled refresh)

---

## 🚀 Future enhancements (post-Store-1)

### Fleet view on landing page

Once Stores 2-3 are live:
- [ ] Add fleet summary KPIs to landing page (total daily sales, fleet DSCR, etc.)
- [ ] Replace placeholder cards with live mini-dashboards for each open store
- [ ] Comparison view: store-vs-store performance

### Store-specific dashboards (Stores 2-10)

- [ ] Template once Taytay is finalized — each new store gets a folder `stores/<name>/` with index.html + data/
- [ ] Should be a quick clone-and-edit per store rather than a rebuild

### Corporate management layer staffing plan

- [ ] Model assumes ₱430K/month at full scale. Need a hiring calendar:
  - When does the ops manager get hired? (4th store opens?)
  - When do area supervisors come in? (6th-7th stores?)
- [ ] Tie staffing tier to store-count milestones

### Additional ops KPIs (when data is available)

The current weekly KPI dashboard could grow to include:
- [ ] Customer return rate (from loyalty/QR data when available)
- [ ] Peak-hour utilization (7-9am bottleneck check)
- [ ] Stockouts / 86'd items frequency and SKU
- [ ] Daily weather correlation (once a season's data exists)
- [ ] Staff turnover by store

### Backup site pipeline

- [ ] Fold Shell Whiteplain, Shell/Seaoil Congressional, Kasiglahan Village into the model as contingencies for Sites 8-10 if current pipeline slips

---

## 📝 Recently completed

- ✅ **DATA_ARCHITECTURE.md v1.0.0 designed and locked** (2026-06-04). Store-first folder layout, monthly operating files, single-array financial files, per-store schemas, tiered lenient validation.
- ✅ Built scaffolded GitHub repo `angelo-madrid/PUC_Dashboard` (June 2026)
- ✅ GitHub Pages enabled, live at https://angelo-madrid.github.io/PUC_Dashboard/
- ✅ Landing page with 10-store roadmap, Taytay LIVE + Stores 2-10 placeholders
- ✅ Skeleton data file at `stores/taytay/data/taytay_data.json`
- ✅ CLAUDE.md created for project context continuity

---

## Changelog

| Version | Date | Author | Notes |
|---|---|---|---|
| 1.0.1 | 2026-06-04 | Angelo + Claude | Marked data architecture design items as completed (now specified in DATA_ARCHITECTURE.md v1.0.0). Updated recently completed section. |
| 1.0.0 | 2026-06-04 | Angelo | Initial backlog. Captures landing page visual review items, May 31 capital call decisions, data architecture proposal (operating vs financial vs shared), and future enhancements. |

# PUC Dashboard — Project Context for Claude

**Version:** 1.0.0
**Last updated:** 2026-06-04 14:30 PHT
**Maintainer:** Angelo Madrid

This file gives Claude (Code or chat) the context to work productively on this project without re-explaining background every session.

> **Versioning convention:** Semantic versioning. **Major** (X.0.0) for structural changes to the project (new repo layout, framework swap). **Minor** (1.X.0) for new sections, new data sources, or significant model changes (e.g., adding Eton Centris in-line economics). **Patch** (1.0.X) for corrections, copy edits, and small additions. Bump the version and timestamp whenever you edit this file, and add an entry to the changelog at the bottom.

---

## What this is

A web dashboard for **Nutrizone Food Corp**, the operating company running a **Pickup Coffee (PUC)** franchise rollout. Tracks daily sales, monthly P&L, debt service, and weekly KPIs across what will be **10 stores by end of 2026**. Currently scaffolded with **Taytay (Store #1)** live; Stores 2–10 are placeholders.

- **Live site:** https://angelo-madrid.github.io/PUC_Dashboard/
- **Repo:** https://github.com/angelo-madrid/PUC_Dashboard (public)
- **Local path:** `~/PUC_Dashboard`
- **Distinct from:** `angelo-madrid/Investment_Dashboard` (separate personal investing project)

---

## People & roles

- **Angelo Madrid** — Strategy & Business Development. Owns the financial model, equity equal third with Vincent and SP Madrid.
- **Vincent** — Store Management & Operations. Owns store-level execution. Equal-third equity.
- **SP Madrid** — Parent company. Provides intercompany debt financing at 6.5% per annum, 24-month tenor per store. Equal-third equity.
- **PUC (Pickup Coffee HQ)** — Master franchisor. Takes 7% royalty on gross sales.

---

## Capital structure

- **Equity:** ₱3M seed (₱1M each from Angelo, Vincent, SP Madrid)
- **Debt:** SP Madrid intercompany facility, 6.5% per annum simple interest, 24-month tenor per store
- **Per-store outlay:** ₱2,521,251 (incl. ₱560K franchise fee + capex + working capital). Eton Centris in-line is ₱4M (different format — flagged for separate modeling).
- **Total fleet capital:** ~₱26M across 10 stores

Each store's debt clock starts at disbursement date (cash out from SP Madrid). Repayment uses **level principal + interest on declining balance**, no grace period.

---

## Unit economics (per actual Taytay data)

- **Variable costs:** 51.2% of gross = 32% COGS + 7% PUC royalty + 7.2% aggregator fees + 5% wastage/maintenance reserve
- **Fixed monthly opex:** ₱147,500 (rent 20K + CUSA 2.5K + POS rent 2K + salaries 100K + internet 3K + utilities 10K + marketing 5K + other 5K)
- **Normalized EBITDA margin:** ~30–32% at mature run-rate
- **Mature dry-weather run-rate:** ~₱28-31K/day gross (May 2026 actual: ₱28,310/day full month, ₱31,468/day last 7 days)
- **Recovery thresholds (with allocated mgmt overhead):**
  - 18-month (Stretch): ₱22,764/day
  - 24-month (Sweet Spot, recommended): ₱20,401/day
  - 30-month (Comfort): ₱18,984/day
  - 36-month (Conservative): ₱18,040/day

---

## Seasonality model

Manila has a strong wet/dry pattern. Modeled monthly factors applied to dry-weather baseline:

| Month | Factor | Notes |
|---|---|---|
| Jan | −8% | Post-holiday slump |
| Feb–Apr | 0% | Dry baseline |
| May | −5% | Late dry season |
| Jun | −15% | Monsoon onset |
| Jul | −25% | Wet season |
| Aug | −30% | Peak monsoon |
| Sep | −25% | Wet season |
| Oct | −10% | Late wet / BER kickoff |
| Nov | +8% | Dry, pre-holiday |
| Dec | +12% | Holiday peak (13th month pay) |

**Blended annual factor:** −8.2%. Note: these are anchored on June 1–3 actuals (-30% drop) + climatology; will recalibrate with more data.

---

## Corporate management layer (above stores)

Phases in with store count:
- 1–3 stores: ₱150K/month (founders + 1 supervisor)
- 4–6 stores: ₱280K/month (add ops manager + finance)
- 7–10 stores: ₱430K/month (full layer: ops mgr, 2 area supervisors, finance/admin, marketing, procurement, office, software, ₱100K founder draws Angelo+Vincent)

Annual at full scale: ₱5.16M.

---

## Store rollout schedule

1. **Taytay Diversion** — LIVE since Feb 13, 2026
2. **Eton Centris (in-line)** — Target Jun 30, 2026
3. **Unioil BF Resort** — Target Jun 5, 2026 (strongest pipeline: 12/14 gates done)
4. **Unioil Amang Rodriguez (Pasig)** — Target Jun 13, 2026 (AT RISK — lease starts July 1, after target)
5. **Unioil SLEX Makati** — Target Jun 12, 2026 (lease starts June 15, after target)
6. **Unioil Makati Buendia** — Pending lease
7. **PLDT** — AT RISK (award not started)
8. **East Service Road** — Target Jun 25, 2026
9. **BF Resort Drive** — Target Jun 15, 2026
10. **Smart IOC Building** — Pending

Backup sites awaiting PUC: Shell Whiteplain, Shell Congressional Ext E/W, Seaoil Congressional Ave Ext, Kasiglahan Village kiosk.

---

## Key financial milestones

- **May 31, 2026:** Capital call CC-2026-001 — ₱14.1M for first 5 stores (incl. ₱2.57M Taytay reimbursement w/ ₱53,430 interest)
- **Sep 2026:** All 10 stores expected open
- **Oct 2026:** Peak monthly debt service ₱1.24M (all 10 loans active)
- **Sep 2028:** Fleet fully debt-free
- **2030 (Year 5):** Corporate EBITDA ~₱27.9M. Equity value at 4× = ₱111.6M (₱37.2M per partner).

---

## Tech stack

- Pure HTML/CSS/JS (no build step)
- Chart.js via CDN
- GitHub Pages from `main` branch root
- Manual data updates via JSON files (will eventually automate)

### Design language

- **Fonts:** DM Serif Display (headlines), DM Sans (body), DM Mono (numbers)
- **Colors:** Cream `#faf8f4` background, ink `#0f0e0c` text, green `#1a6b45` accent, terracotta accent on landing page
- **Aesthetic:** Editorial, refined, restrained. No emoji clutter. Tables and small-multiples over big chart-junk.

---

## Conventions & principles

- **Bootstrap mode.** No external investors. Move fast, keep costs lean, validate before scaling.
- **Honesty over optimism.** When data conflicts with assumptions (e.g., June rain dropping sales), reflect it in the model rather than smoothing it over.
- **Per-store first, fleet second.** Each store stands on its own economics. Fleet model is the sum of stores, not a top-down aggregate.
- **Memo over slides.** When summarizing, prefer prose with embedded tables/charts to bulleted decks.
- **Numbers in PHP (₱).** Use comma separators. Show full numbers in tables, abbreviated (M/K) in headlines.

---

## What NOT to do without confirmation

- Don't paste the Taytay dashboard HTML into `stores/taytay/index.html` until explicitly asked (separate session for that).
- Don't change cost assumptions, seasonality factors, or capital structure parameters without flagging.
- Don't make the repo private — Pages needs public for free deployment.
- Don't add a `LICENSE` file — this is proprietary internal use.

---

## Changelog

| Version | Date | Author | Notes |
|---|---|---|---|
| 1.0.0 | 2026-06-04 | Angelo | Initial project context. Captures capital structure, unit economics, seasonality, rollout schedule, tech stack, design language, and conventions as of repo scaffold completion. |

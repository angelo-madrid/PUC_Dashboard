# PUC Dashboard — Data Architecture Specification

**Version:** 1.0.0
**Last updated:** 2026-06-04 16:00 PHT
**Maintainer:** Angelo Madrid
**Status:** Locked — ready to build

This spec defines how data is structured, owned, and validated across the PUC Dashboard project. It exists so any future Claude Code session (or human collaborator) can build, edit, or extend data files without re-deriving design decisions.

---

## Principles

1. **Source-of-truth clarity.** Each data point has exactly one canonical home. Daily sales live in operating files only. Monthly P&L derives from daily aggregates rather than duplicating them.
2. **Human-readable, machine-validatable.** Vincent edits operating JSON via GitHub web UI. Field names are plain English. The dashboard JS validates on load and surfaces issues without blocking.
3. **Ownership and cadence visibility.** Every file declares its owner and update cadence in its `_meta` block.
4. **Tolerate noise, surface signal, never block.** Validation issues warn but never prevent the dashboard from loading.

---

## Folder structure

```
data/
├── taytay/
│   ├── operating/
│   │   ├── 2026-02.json        ← Vincent appends daily, closes monthly
│   │   ├── 2026-03.json
│   │   ├── 2026-04.json
│   │   ├── 2026-05.json
│   │   └── 2026-06.json        ← Currently in_progress
│   ├── financial/
│   │   ├── pnl.json            ← All months in one array
│   │   ├── debt_schedule.json  ← Static, set at origination
│   │   └── budget.json         ← Annual, locked
│   └── _schemas/
│       ├── operating_daily.json
│       └── financial_pnl.json
├── shared/
│   ├── cost_assumptions.json
│   └── seasonality_factors.json
└── README.md
```

**Layout rationale:** store-first organization mirrors the dashboard route structure (`/stores/taytay/`). Each store is self-contained: operating, financial, schemas. Model parameters that apply across all stores live in `/data/shared/`.

**Monthly file rationale:** high-frequency operating data is split by calendar month rather than one giant array. Vincent edits one short file per week. Closed months become append-only. Git diffs stay scoped.

---

## Cross-cutting conventions

| Convention | Format |
|---|---|
| Currency | Plain number in Philippine Peso. No symbol, no thousand separators. `28310.50` not `"₱28,310.50"`. Format on display. |
| Percentages | Decimal form. `0.32` not `32%` or `"32%"`. |
| Dates | ISO 8601 — `2026-06-01` for dates, `2026-06-04T14:30:00+08:00` for timestamps. PHT timezone explicit. |
| `null` | "Not yet recorded" or "deliberately omitted." |
| Missing field | "Doesn't apply to this record." |
| `0` | "Recorded as zero." |
| Version bumps | Each file has `schema_version` in `_meta`. Bump when the *structure* changes, not when content changes. |

---

## File specs

### `data/taytay/operating/YYYY-MM.json`

One file per calendar month. Vincent's primary update target.

```json
{
  "_meta": {
    "store_id": "taytay",
    "store_name": "Taytay Diversion",
    "data_type": "operating_daily",
    "month": "2026-06",
    "status": "in_progress",
    "days_recorded": 4,
    "days_in_month": 30,
    "last_entry_date": "2026-06-04",
    "owner": "Vincent (Operations)",
    "update_cadence": "weekly — every Monday for prior week",
    "source": "EOD reports from store, validated against Mosaic POS",
    "schema_version": "1.0.0",
    "last_updated": "2026-06-04T16:00:00+08:00"
  },
  "daily": [
    {
      "date": "2026-06-01",
      "weekday": "Mon",
      "gross_sales": 20104.99,
      "transaction_count": 100,
      "cups_sold": {
        "Y32": 9,
        "Y22": 84,
        "Y16": 105,
        "H12": 12,
        "H8": 9,
        "total": 219
      },
      "channels": {
        "cash": 8779.28,
        "pickup_app": 3138.00,
        "grab_food": 4162.14,
        "foodpanda": 2579.00,
        "maya_qrph": 1446.57,
        "maya_credit": 0,
        "maya_debit": 0
      },
      "operations": {
        "barista_count": 3,
        "supervisor_count": 1,
        "total_worked_hours": 32,
        "overtime_hours": 0,
        "kwh_reading": 4267.5,
        "ice_expense": 235,
        "transpo_expense": 0,
        "other_expense": 0
      },
      "quality": {
        "voids_count": 0,
        "voids_value": 0,
        "wastage_count": 1,
        "wastage_note": ""
      },
      "weather": "rainy",
      "operator_notes": "Slow walking and aggregator due to bad weather. Unioil also has few customers.",
      "reported_by": "sunshine"
    }
  ]
}
```

**Field notes:**
- `status`: `in_progress` (active month) or `closed` (no more edits expected). Set by Angelo at month-end.
- `net_sales` is intentionally absent from operating files. Net is derived from gross by finance — single source of truth.
- `cups_sold.total` is redundant with the sum of cup sizes but validated against. If they don't match, the file is wrong.
- `weather` is a controlled vocabulary: `clear` | `cloudy` | `rainy` | `stormy` | `typhoon`.
- `operator_notes` is free-form text — preserved verbatim from EOD reports for context.

### `data/taytay/_schemas/operating_daily.json`

```json
{
  "schema_version": "1.0.0",
  "applies_to": "data/taytay/operating/*.json",
  "required_fields": [
    "date", "weekday", "gross_sales", "transaction_count",
    "cups_sold", "channels", "operations", "quality", "reported_by"
  ],
  "validations": {
    "gross_sales": {
      "type": "number",
      "min": 0,
      "max": 100000,
      "sanity_warn_below": 10000,
      "sanity_warn_above": 50000
    },
    "transaction_count": {
      "type": "integer",
      "min": 0,
      "max": 500,
      "sanity_warn_below": 50
    },
    "cups_sold_total_check": {
      "rule": "cups_sold.total must equal sum of Y32 + Y22 + Y16 + H12 + H8",
      "tolerance": 0,
      "on_fail": "warn"
    },
    "channels_sum_check": {
      "rule": "sum of all channels must equal gross_sales",
      "tolerances": {
        "silent": 1,
        "log": 50,
        "row_warn": 500,
        "banner_warn": null
      },
      "on_fail": "warn (never block load)"
    },
    "date": {
      "type": "ISO 8601 date (YYYY-MM-DD)",
      "rule": "must not be in the future, must fall within the file's month"
    },
    "weather": {
      "type": "enum",
      "values": ["clear", "cloudy", "rainy", "stormy", "typhoon"]
    },
    "weekday": {
      "type": "enum",
      "values": ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"]
    }
  }
}
```

### `data/taytay/financial/pnl.json`

Low-frequency, one file for all monthly P&L records.

```json
{
  "_meta": {
    "store_id": "taytay",
    "data_type": "financial_pnl",
    "owner": "Angelo (Finance)",
    "update_cadence": "monthly — first week of following month",
    "source": "BIR Sales Summary + daily aggregation + actual cost recording",
    "schema_version": "1.0.0",
    "last_updated": "2026-06-04T16:00:00+08:00"
  },
  "months": [
    {
      "month": "2026-05",
      "days_in_month": 31,
      "gross_sales": 877625.44,
      "net_sales": 745420.00,
      "variable_costs": {
        "cogs": 280840.14,
        "puc_royalty": 61433.78,
        "aggregator_fees": 63188.95,
        "wastage_maintenance": 43881.27,
        "total": 449344.14
      },
      "fixed_costs": {
        "rent": 20000,
        "cusa": 2500,
        "pos_rent": 2000,
        "salaries": 100000,
        "internet": 3000,
        "utilities": 10000,
        "marketing": 5000,
        "other": 5000,
        "total": 147500
      },
      "ebitda": 280781.30,
      "ebitda_margin": 0.320,
      "debt_service": {
        "reimbursement_interest": 53430,
        "principal_repayment": 0,
        "ongoing_interest": 0,
        "total": 53430
      },
      "net_income": 227351.30,
      "notes": "Reimbursement of equity advance settled May 31. New 24-month loan booked simultaneously."
    }
  ]
}
```

### `data/taytay/financial/debt_schedule.json`

Static, set at loan origination. Updated only on refinancing.

```json
{
  "_meta": {
    "store_id": "taytay",
    "data_type": "debt_schedule",
    "owner": "Angelo (Finance)",
    "update_cadence": "static — set at origination, updated only on refinancing",
    "source": "Loan agreement CDV-2026-001",
    "schema_version": "1.0.0",
    "last_updated": "2026-06-04T16:00:00+08:00"
  },
  "loan_terms": {
    "principal": 2521251,
    "interest_rate_annual": 0.065,
    "tenor_months": 24,
    "method": "level principal + interest on declining balance",
    "grace_period_months": 0,
    "disbursement_date": "2026-05-31",
    "first_payment_date": "2026-06-30",
    "final_payment_date": "2028-05-31",
    "lender": "SP Madrid",
    "borrower": "Nutrizone Food Corp"
  },
  "schedule": [
    {
      "period": 0,
      "date": "2026-05-31",
      "opening_balance": 0,
      "principal_payment": 0,
      "interest_payment": 53430,
      "interest_type": "reimbursement",
      "notes": "One-time, equity advance settlement",
      "total_payment": 53430,
      "ending_balance": 2521251
    }
  ]
}
```

### `data/taytay/financial/budget.json`

Annual budget. Locked at start of year.

```json
{
  "_meta": {
    "store_id": "taytay",
    "data_type": "budget",
    "owner": "Angelo (Finance)",
    "update_cadence": "annual — set in January, locked thereafter",
    "schema_version": "1.0.0",
    "last_updated": "2026-06-04T16:00:00+08:00"
  },
  "year": 2026,
  "budget_basis": "original opening plan, linear +5% MoM ramp from ₱18K/day Feb baseline",
  "months": [
    {
      "month": "2026-02",
      "days": 16,
      "daily_budget_gross": 18000,
      "monthly_budget_gross": 288000
    },
    {
      "month": "2026-03",
      "days": 31,
      "daily_budget_gross": 18900,
      "monthly_budget_gross": 585900
    }
  ]
}
```

### `data/shared/cost_assumptions.json`

Model parameters — referenced by financial calculations across all stores.

```json
{
  "_meta": {
    "data_type": "model_parameters",
    "owner": "Angelo (Finance)",
    "update_cadence": "as needed — when fundamentals change",
    "applies_to": "all stores (cart format)",
    "schema_version": "1.0.0",
    "last_updated": "2026-06-04T16:00:00+08:00"
  },
  "variable_cost_percentages": {
    "cogs": 0.32,
    "puc_royalty": 0.07,
    "aggregator_fees_blended": 0.072,
    "wastage_maintenance_reserve": 0.05,
    "total": 0.512
  },
  "fixed_costs_monthly": {
    "rent": 20000,
    "cusa": 2500,
    "pos_rent": 2000,
    "salaries": 100000,
    "internet": 3000,
    "utilities": 10000,
    "marketing": 5000,
    "other": 5000,
    "total": 147500
  },
  "calibration_notes": "Based on Taytay May 2026 actuals. Recalibrate after 90+ days of multi-store data."
}
```

### `data/shared/seasonality_factors.json`

Monthly multipliers applied to dry-weather baseline.

```json
{
  "_meta": {
    "data_type": "seasonality_model",
    "owner": "Angelo (Finance)",
    "update_cadence": "calibrate quarterly with actual data",
    "applies_to": "all stores",
    "schema_version": "1.0.0",
    "last_updated": "2026-06-04T16:00:00+08:00"
  },
  "factors": {
    "01": { "factor": -0.08, "driver": "post-holiday slump" },
    "02": { "factor":  0.00, "driver": "dry baseline" },
    "03": { "factor":  0.00, "driver": "dry baseline" },
    "04": { "factor":  0.00, "driver": "dry baseline" },
    "05": { "factor": -0.05, "driver": "pre-monsoon transition" },
    "06": { "factor": -0.15, "driver": "monsoon onset" },
    "07": { "factor": -0.25, "driver": "wet season" },
    "08": { "factor": -0.30, "driver": "peak monsoon" },
    "09": { "factor": -0.25, "driver": "wet season" },
    "10": { "factor": -0.10, "driver": "late wet / BER kickoff" },
    "11": { "factor":  0.08, "driver": "dry, pre-holiday" },
    "12": { "factor":  0.12, "driver": "holiday peak (13th month pay)" }
  },
  "blended_annual": -0.082,
  "calibration_basis": "June 1-3, 2026 actuals (-30% drop) + Manila climatology",
  "next_recalibration": "September 2026 (post-monsoon)"
}
```

---

## Validation behavior

The dashboard's data loader (client-side JS) validates each file on fetch. Validation failures **never block the dashboard from rendering** — they surface as warnings instead.

### Tolerance bands for channel-sum check

Daily channel totals should equal `gross_sales`. Mismatches handled as:

| Mismatch | Behavior |
|---|---|
| ≤ ₱1 | Silent — rounding noise |
| ₱1 – ₱50 | Log to console only |
| ₱50 – ₱500 | Show ⚠ icon next to the row |
| > ₱500 | Show banner at top of dashboard: "Channel mismatch ₱X on YYYY-MM-DD — please verify" |

This pattern applies across all files: noisy on big problems, quiet on small ones, never silent on real ones, never blocking on any.

### Schema version check

If a file's `schema_version` doesn't match what the dashboard expects, show a banner: "This file uses schema vX.X.X; dashboard expects vY.Y.Y. Data may render incorrectly."

---

## Update workflow (preview)

This will be expanded in a separate operations document, but in summary:

| Cadence | Owner | Files touched |
|---|---|---|
| Weekly (Mondays) | Vincent | Current month's `operating/YYYY-MM.json` — append last week's daily records |
| Monthly (first week) | Angelo | `financial/pnl.json` — add closed month; mark prior `operating/YYYY-MM.json` as `closed` |
| Quarterly | Angelo | `shared/seasonality_factors.json` — recalibrate if data warrants |
| Per loan event | Angelo | `financial/debt_schedule.json` — only on refinancing |
| Annual | Angelo | `financial/budget.json` — set Jan 1, locked |

---

## Open questions deferred to future versions

- **Multi-currency support.** Eton Centris may eventually want USD-denominated reporting if expansion path includes a regional view. Not needed v1.0.
- **POS-direct integration.** Mosaic POS API integration would let the operating files self-update. Tracked separately in backlog.
- **Audit history.** Currently relies on Git history. If finer-grained audit (who edited which field when) becomes important, may need a separate ledger.

---

## Changelog

| Version | Date | Author | Notes |
|---|---|---|---|
| 1.0.0 | 2026-06-04 | Angelo + Claude | Initial locked architecture. Store-first folder layout, monthly operating files, single-array financial files, per-store schemas, lenient channel-sum validation with tiered tolerances. |

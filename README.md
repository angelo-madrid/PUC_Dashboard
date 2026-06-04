# PUC Dashboard

Pickup Coffee franchise performance dashboard — **Nutrizone Food Corp**.
Daily sales, P&L, debt service, and weekly KPI tracking across the store network.

**Live:** https://angelo-madrid.github.io/PUC_Dashboard/

---

## About

- **Operator:** Nutrizone Food Corp
- **Parent:** SP Madrid
- **Franchise master:** Pickup Coffee (PUC)
- **Network goal:** 10 stores live by end of 2026, added progressively.

Each store has its own self-contained dashboard under `stores/<store>/`. The
landing page (`index.html`) is a store selector linking to each live dashboard.

## Folder structure

```
PUC_Dashboard/
├── index.html                   # Landing page — store selector
├── stores/
│   └── taytay/
│       ├── index.html           # Taytay dashboard (data embedded inline for now)
│       └── data/
│           └── taytay_data.json # Daily sales, P&L, debt service, weekly KPI
├── assets/
│   └── shared.css               # Optional shared design tokens
├── .gitignore
└── README.md
```

## Updating Taytay data

The Taytay dashboard currently has its data **embedded inline** in
`stores/taytay/index.html`. For now, edit that file directly.

Later, data will migrate to `stores/taytay/data/taytay_data.json`. Once
migrated, the update flow is:

1. Edit `stores/taytay/data/taytay_data.json` (append the latest daily figures).
2. Update the `meta.last_updated` field.
3. Commit and push to `main` — GitHub Pages redeploys automatically.

```bash
git add stores/taytay/data/taytay_data.json
git commit -m "Update Taytay data — <date>"
git push
```

## Stores planned

| # | Store              | Status   |
|---|--------------------|----------|
| 1 | Taytay             | Live     |
| 2 | Eton Centris       | Planned  |
| 3 | BF Resort          | Planned  |
| 4 | Amang Rodriguez    | Planned  |
| 5 | SLEX Makati        | Planned  |
| 6 | Makati Buendia     | Planned  |
| 7 | PLDT               | Planned  |
| 8 | East Service Road  | Planned  |
| 9 | BF Resort Drive    | Planned  |
| 10| Smart IOC Building | Planned  |

## Tech stack

- Pure HTML / CSS / JavaScript — no build step.
- [Chart.js](https://www.chartjs.org/) via CDN for charts.
- Fonts: DM Serif Display + DM Sans (Google Fonts).
- Hosted on GitHub Pages (deploys from `main`).

## Contact

**Angelo Madrid** — Nutrizone Food Corp

---

_Proprietary — internal use. No license granted._

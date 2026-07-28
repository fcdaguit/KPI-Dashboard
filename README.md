# FAST Regional Ops KPI Dashboard

A static, GitHub-Pages-deployable KPI dashboard for the Regional Operations
Manager, consolidating sellout performance across multiple principals
(Shell Lubricants, Monde Nissin, CDO Food, and future suppliers).

No backend, no build step — plain HTML/CSS/JS + Chart.js, reading a single
JSON file that's refreshed from each principal's Google Sheet export.

## How it works

```
Principal IT admin fills in     →  Google Sheet
their monthly Sellout/Target       (Branch/Site, Principal/Supplier,
data using the shared template     Channel, Category, Month, Year,
                                    Target Sales, Actual Sales)
        │
        │  File ▸ Download ▸ CSV
        ▼
raw/<principal>-<month>.csv
        │
        │  python3 scripts/build_data.py raw/*.csv
        ▼
data/sales-data.json   ← the dashboard reads this file directly
        │
        │  git commit + push
        ▼
GitHub Pages  (auto-redeploys the static site)
```

There is no live API call to Google Sheets — this keeps the site static,
free to host, and independent of any principal's sheet-sharing permissions.
The trade-off is a manual (or scripted/scheduled) export step each
reporting period.

## Project structure

```
index.html              Page shell — header, filters, panels, footer
css/styles.css           FAST brand tokens (blue #0057A0 / red #D62828), layout
js/app.js                 Data loading, filtering, charts, heatmap, recommendations
data/sales-data.json      Combined dataset the dashboard reads (sample data included)
data/template.csv         Header template to hand to each principal's IT admin
scripts/build_data.py     Combines raw CSV exports into data/sales-data.json
raw/                       Drop new monthly CSV exports here (not committed by default)
```

## Local preview

```bash
python3 -m http.server 8000
# open http://localhost:8000
```

(Opening `index.html` directly via `file://` will not work — the browser
blocks `fetch()` of local JSON files without a server.)

## Monthly refresh workflow

1. Send `data/template.csv` to each principal's IT admin; they fill in one
   row per Branch/Site × Channel × Category × Month.
2. Download each principal's sheet as CSV into a local `raw/` folder.
3. Run:
   ```bash
   python3 scripts/build_data.py raw/*.csv
   ```
   This validates headers and rewrites `data/sales-data.json`.
4. Commit and push:
   ```bash
   git add data/sales-data.json
   git commit -m "Refresh sales data — <month year>"
   git push
   ```
5. GitHub Pages redeploys automatically (usually within ~1 minute). Anyone
   viewing the live site can also press **Refresh** in the header to
   re-fetch the latest committed JSON without a hard reload.

## Deploying to GitHub Pages

1. Create a new GitHub repository and push this folder's contents to it:
   ```bash
   git init
   git add .
   git commit -m "Initial KPI dashboard"
   git branch -M main
   git remote add origin https://github.com/<your-org>/<repo-name>.git
   git push -u origin main
   ```
2. In the repo, go to **Settings → Pages**.
3. Under **Build and deployment**, set **Source** to `Deploy from a branch`,
   branch `main`, folder `/ (root)`. Save.
4. GitHub gives you a URL like
   `https://<your-org>.github.io/<repo-name>/` — that's the dashboard link
   to share with the Regional Operations Manager.
5. Every future `git push` to `main` (e.g. after a data refresh) redeploys
   the site automatically — no extra CI configuration needed for a static
   site like this.

## Adding a new principal / supplier

The dashboard has no hard-coded list of principals, branches, channels, or
categories — every filter is derived from whatever is present in
`data/sales-data.json`. To onboard a new supplier:

1. Share `data/template.csv` with their IT admin.
2. Add their exported CSV to `raw/` and re-run `build_data.py`.
3. Nothing in `index.html`, `app.js`, or `styles.css` needs to change.

## Notes on this build

- **Sample data**: `data/sales-data.json` currently ships with realistic
  generated sample data (Jan 2025 – Jul 2026) across the three named
  principals so the dashboard is fully functional out of the box. Replace
  it with real exports via `build_data.py` whenever you're ready.
- **Charts**: Chart.js is loaded from a CDN (`cdn.jsdelivr.net`) — no
  npm install or bundler required, which keeps GitHub Pages deployment a
  plain static-file push.
- **Export to PDF/Excel/PowerPoint** (mentioned in the original spec) is
  not yet wired up — the cleanest static-site approach is a "Print" /
  Save-as-PDF via the browser for now; a dedicated export button can be
  added next (e.g. via `SheetJS` for Excel, `jsPDF` for PDF).

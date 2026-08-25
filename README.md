# Riverkeep Trust — NbS Impact Dashboard

A public-facing dashboard communicating the impact of nature-based river restoration work along the (fictional) Nkosana River in Rivermead. Built as a portfolio piece modeled on a real-world consultancy RFP for a public NbS communication dashboard.

**Live site:** https://riverkeep-trust.netlify.app

All data in this project is fictional, built for demonstration purposes only.

---

## What this is

A narrative-led, indicator-driven public dashboard (in the style of [MyPeg](https://www.mypeg.ca)) showing restoration progress across four river sites, split into three themes:

- **Flood Resilience** — riparian corridor restoration, debris removal, flood extent reduction
- **River Health** — water quality, native species, vegetated cover, urban tree planting
- **Community & Livelihoods** — jobs created, women's-group participation

It combines a custom-coded front end with an embedded Power BI report for deeper drill-down.

---

## Architecture

- **Front end:** single static `index.html` — no framework, no build step. HTML/CSS/vanilla JS.
- **Charts:** [Chart.js](https://www.chartjs.org/) (line/bar), loaded via CDN
- **Map:** [Leaflet.js](https://leafletjs.com/) with OpenStreetMap tiles, loaded via CDN
- **Data:** [PapaParse](https://www.papaparse.com/) fetches and parses `riverkeep_indicators.csv` client-side at page load — every stat, chart, and map marker is derived from that file at runtime, nothing is hardcoded in the HTML/JS
- **Deeper analysis:** a companion Power BI report, published via "Publish to web" and embedded as an iframe in the "For Funders" section
- **Hosting:** static site, deployed on Netlify from this repo's `main` branch (auto-redeploys on push)

No backend, no database — this is intentional, matching the MVP-scope constraints of the RFP this project is modeled on (no real-time data, no advanced analytics, communication-first).

---

## Repo structure

```
├── index.html                      # the dashboard (this is the whole site)
├── riverkeep_indicators.csv        # clean, analysis-ready data — what index.html reads
├── riverkeep_indicators_za.csv     # same data, semicolon-delimited / comma-decimal
│                                    #   (for Power BI imports using en-ZA locale)
└── raw_data/
    ├── DATA_DICTIONARY.md          # full field-by-field documentation of every file below
    ├── site_register.csv           # master site metadata
    ├── field_monitoring_log.csv    # raw per-visit restoration data
    ├── water_quality_lab_results.csv
    ├── species_survey_log.csv
    ├── vegetation_cover_survey.csv
    ├── employment_records.csv
    ├── community_participation_log.csv
    └── hydrology_readings.csv
```

`raw_data/` holds the source files each clean indicator was aggregated from — see
`raw_data/DATA_DICTIONARY.md` for the full lineage (which raw column feeds which clean indicator,
and what transformation was applied).

---

## Running it locally

Because `index.html` fetches `riverkeep_indicators.csv` via JavaScript, it **must be served over HTTP** — opening the file directly (`file://`) will fail due to browser CORS restrictions on local file reads.

```bash
git clone https://github.com/<your-username>/riverkeep-trust.git
cd riverkeep-trust
python3 -m http.server 8000
```

Then open `http://localhost:8000`.

---

## Data pipeline

1. **Raw data** (`raw_data/*.csv`) — one file per source system (field logs, lab results, HR records, etc.), documented in `raw_data/DATA_DICTIONARY.md`
2. **Cleaning** — raw files are aggregated (summed/grouped by site, indicator, and quarter) into the single long-format `riverkeep_indicators.csv`
3. **Presentation** — `index.html` fetches that clean CSV at runtime and computes every hero stat, chart, and map marker from it directly

Each row in `riverkeep_indicators.csv` also carries a `Metric_Type` field (`cumulative` or `point_in_time`), which determines whether a value is safe to treat as a running total or must be read as a standalone snapshot — see the data dictionary for details.

---

## Power BI report

The embedded report in the "For Funders" section was built in Power BI Desktop from `riverkeep_indicators.csv`, then published via **File → Publish to web**. That method is fully public (no login) and free — no Power BI Pro/Premium/Embedded capacity subscription required, which matches this project's zero-budget MVP framing.

---

## Known limitations

- Data updates require manually replacing the CSV and redeploying — no live data feed (by design, per MVP scope)
- The Power BI embed and the custom map are two separate visualizations reading the same source data independently; styling one doesn't automatically update the other
- isiZulu text throughout is illustrative/placeholder, not professionally translated

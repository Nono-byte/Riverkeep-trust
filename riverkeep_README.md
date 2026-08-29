# Riverkeep Trust, Nature based Solutions Impact Dashboard

A public facing dashboard communicating the impact of nature based river restoration work along the Nkosana River in Rivermead. Built as a portfolio piece, modelled on a real world consultancy RFP for a public NbS communication dashboard.

**Live site:** https://riverkeep-trust.netlify.app

All data in this project is fictional and generated for demonstration purposes. No real site, beneficiary, employment or water quality data appears anywhere in this repository.

---

## What this is

A narrative led, indicator driven public dashboard showing restoration progress across four river sites, organised into three themes:

| Theme | Indicators |
|---|---|
| Flood Resilience | Riparian corridor restoration, debris removal, flood extent reduction |
| River Health | Water quality, native species, vegetated cover, urban tree planting |
| Community and Livelihoods | Jobs created, women's group participation |

It combines a custom coded front end with an embedded Power BI report for deeper drill down.

---

## Indicator definitions and data lineage

This is the part of the project that took the most work, and the part that determines whether the numbers on the dashboard are correct.

Every clean indicator traces back to a named raw source. `raw_data/DATA_DICTIONARY.md` documents, field by field, which raw column feeds which clean indicator and what transformation was applied to get there. Nothing on the dashboard is a number without a documented origin.

### Cumulative versus point in time

Every row in `riverkeep_indicators.csv` carries a `Metric_Type` field, set to either `cumulative` or `point_in_time`. This determines whether a value may be treated as a running total or must be read as a standalone snapshot.

The distinction matters more than it first appears. Trees planted is cumulative, so summing across quarters is correct. Water quality index is point in time, so summing it produces a meaningless number that still renders perfectly well on a chart. Without the field carried explicitly on every row, that error is invisible: the dashboard looks right and the figure is wrong.

Making it an attribute of the data rather than a rule held in the presentation layer means the constraint travels with the record into Power BI, into any future export, and into whatever anyone builds from the CSV next.

### Aggregation path

```
raw_data/*.csv                one file per source system
        |                     field logs, lab results, HR records, participation logs
        v
   aggregation                summed or grouped by site, indicator and quarter
        |
        v
riverkeep_indicators.csv      single long format table, one row per site,
        |                     indicator and quarter, carrying Metric_Type
        v
   index.html                 fetches the clean CSV at runtime, computes every
                              hero stat, chart and map marker from it directly
```

Nothing is hardcoded in the HTML or JavaScript. Replacing the CSV changes the entire dashboard.

---

## Architecture

| Layer | Choice |
|---|---|
| Front end | Single static `index.html`. No framework, no build step. HTML, CSS, vanilla JS |
| Charts | Chart.js, line and bar, via CDN |
| Map | Leaflet.js with OpenStreetMap tiles, via CDN |
| Data loading | PapaParse fetches and parses `riverkeep_indicators.csv` client side at page load |
| Deeper analysis | Companion Power BI report, published via Publish to web, embedded as an iframe in the For Funders section |
| Hosting | Static site on Netlify, deployed from `main`, auto redeploys on push |

No backend and no database. This is a deliberate scoping decision rather than a shortcut, and it matches the MVP constraints of the RFP the project is modelled on: communication first, no real time data, no advanced analytics. Adding a backend would have introduced hosting cost, a maintenance burden and an authentication surface, none of which the brief called for.

---

## Repo structure

```
├── index.html                      # the dashboard, this is the whole site
├── riverkeep_indicators.csv        # clean, analysis ready data, what index.html reads
├── riverkeep_indicators_za.csv     # same data, semicolon delimited with comma decimals,
│                                   #   for Power BI imports under en-ZA locale
└── raw_data/
    ├── DATA_DICTIONARY.md          # field by field documentation of every file below
    ├── site_register.csv           # master site metadata
    ├── field_monitoring_log.csv    # raw per visit restoration data
    ├── water_quality_lab_results.csv
    ├── species_survey_log.csv
    ├── vegetation_cover_survey.csv
    ├── employment_records.csv
    ├── community_participation_log.csv
    └── hydrology_readings.csv
```

The second CSV exists because Power BI under an en-ZA locale reads comma decimals and semicolon delimiters, and importing the standard file silently misparses numeric columns. It is a small thing that costs an afternoon to diagnose if it is not handled up front.

---

## Running it locally

`index.html` fetches the CSV via JavaScript, so it must be served over HTTP. Opening the file directly with `file://` will fail on browser CORS restrictions for local file reads.

```bash
git clone https://github.com/Nono-byte/riverkeep-trust.git
cd riverkeep-trust
python3 -m http.server 8000
```

Then open http://localhost:8000.

---

## Power BI report

The embedded report in the For Funders section was built in Power BI Desktop from `riverkeep_indicators.csv`, then published using File, Publish to web.

That publishing method is fully public with no login and requires no Pro, Premium or Embedded capacity, which fits the zero budget MVP framing. The trade off is that Publish to web is genuinely public to anyone with the link and carries no row level security, so it is appropriate here only because every figure in this project is fictional. It would not be appropriate for real beneficiary or employment data.

---

## Known limitations

**Manual data updates.** Refreshing the dashboard means replacing the CSV and redeploying. There is no live feed, by design, per the MVP scope.

**Two independent visualisation layers.** The Power BI embed and the custom map read the same source data separately, so styling one does not update the other. Acceptable at this scope, and the first thing I would consolidate if the project moved past MVP.

**isiZulu strings are not yet reviewed by a first language speaker.** The isiZulu text throughout the interface is currently placeholder and has not been checked for accuracy or register. It is marked as such in the interface so that no reader is misled into treating it as a finished translation. Serving a South African audience partly in isiZulu was a deliberate design decision, and shipping unreviewed translation would undercut the point of making it.

---

## Context

Built as a portfolio piece to work through a specific problem: how a small conservation trust with no data infrastructure and no budget communicates measurable impact to funders and to the public at the same time. Those two audiences want different things from the same dataset, which is why the site carries a narrative public view and a drill down funder view over a single source of truth.

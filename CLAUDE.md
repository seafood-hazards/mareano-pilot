# CLAUDE.md

Quarto static website for the Mareano Pilot Database, deployed to GitHub Pages. Pilot study for submitting Norwegian marine sediment trace element data to EFSA.

## Build

```r
renv::restore()   # restore R packages
quarto render     # renders site to _site/
```

`download_resources.R` runs automatically as a Quarto pre-render step — downloads `pilot_mareano.sqlite` and sql.js/stratum-sqlite libraries into `libs/sqljs/` if missing.

CI/CD: `.github/workflows/publish.yml` — triggers on push to `main`, renders and deploys `_site/` to GitHub Pages.

## Git Workflow

Gitflow. No pull requests — merge feature branches directly and delete them when done.

## Stack

- **Quarto** — static site generator; R chunks for tables, OJS blocks for interactivity
- **R** — tibble, dplyr, knitr (table rendering only; no server-side data access)
- **Observable JS (OJS)** — client-side reactive queries against SQLite in the browser
- **sql.js + stratum-sqlite** — WebAssembly SQLite engine served from `libs/sqljs/`
- **Leaflet.js** — interactive maps; **Observable Plot / D3** — static plots
- **renv** — R package version pinning (`renv.lock`)

## Database Schema (SQLite)

Six tables; all client-side access via stratum-sqlite.

| Table | PK | Notes |
|---|---|---|
| `cruise` | `cruise_id` | ID format: `MA-{year}-{cruise_no}` |
| `core` | `cruise_id, core_id` | lat/lon, mbsl, dist_to_coast, country, municipality, sea_name |
| `sample` | `cruise_id, core_id, sample_id` | depth_from, depth_to, batch_id |
| `parameter` | `parameter` | element, symbol, unit, method1, method2 |
| `sediment` | `cruise_id, core_id, sample_id, parameter` | value, is_lld (1 = below detection limit) |
| `lld` | `batch_id, parameter` | Lower Level of Detection; many-to-many with sample — never JOIN directly |

## OJS Database Pattern

Every page that queries the DB includes `_db-setup.qmd`, which opens the SQLite file via stratum-sqlite:

```qmd
{{< include _db-setup.qmd >}}
```

`header.html` sets `window._stratumSQLite`, `window._dbPath`, and `window._sqljsBase` at page load. The `db` object is then available in all OJS blocks on that page.

## Pages

| File | Description |
|---|---|
| `index.qmd` | Home — pipeline overview, sediment core count table |
| `db-schema.qmd` | ER diagram + table column definitions |
| `invalid-data.qmd` | Data corrections applied to raw Mareano source |
| `distance-to-coast.qmd` | Distance calculation methods + map (Observable Plot) |
| `location-names.qmd` | Country/municipality/sea name estimation methods |
| `distance-interactive-map.qmd` | Leaflet map, markers color-coded by distance category |
| `data-export.qmd` | Flat TSV export format description |
| `efsa-format.qmd` | EFSA submission column schema and catalogue tables |
| `efsa-submission.qmd` | DB field → EFSA column mapping |
| `pilot-db-viewer.qmd` | Paginated read-only SQLite table browser |
| `sediment-map.qmd` | Leaflet map with element selector and year range filter |

# SQL-First County Earthquake Risk ETL (DuckDB)

A small, SQL-driven ETL pipeline that links earthquake events to U.S. counties by proximity and produces **county-level hazard, exposure, and a simple risk index**.  
Built with **DuckDB + SQL (CTEs)** and a thin Python wrapper for execution + plotting.

> Goal: a transparent “screening tool” we can run end-to-end and inspect at every step.

---

## What this project does

Given:
- **County population centroids** (FIPS, population, lat/lon)
- **Earthquake events** (time, id, magnitude, lat/lon)

This pipeline:

1. **Loads** the raw CSVs into DuckDB staging tables  
2. **Cleans** and standardizes schemas into:
   - `counties`
   - `eq_events`
3. **Links** earthquakes to counties using **Haversine distance**, with a magnitude-dependent radius:
   - `mag < 7.0` → within **200 km**
   - `mag >= 7.0` → within **250 km**
4. Computes county-level:
   - **Raw hazard**: `SUM( mag^1.5 / (1 + distance_km) )`
   - **Raw exposure**: `LOG(1 + population)`
   - **Standardized** (z-score) hazard & exposure
   - **Risk index**: `z_hazard * z_exposure`
5. Generates quick QC/summary plots and tables.

---

## Why “SQL-first”?

All core logic is written as **SQL CTEs** (cleaning, linking, aggregations, z-scores).  
This makes the pipeline:
- easy to audit
- easy to modify
- easy to explain in a report / interview

Python is used mainly to:
- run the SQL blocks
- export plots

---

## Outputs

This script generates:

### Tables (inside DuckDB)
- `counties`
- `eq_events`
- `event_county_links`
- `qc_link_summary`
- `county_hazard_risk`
- `state_risk_summary`

### Figures (saved as PNG)
- `qc_distance_histogram.png` — distribution of county–event link distances  
- `top30_counties_risk_bar.png` — top 30 counties by risk index  
- `haz_vs_expo_scatter.png` — hazard vs exposure colored by risk index  

---

## Project report

- `docs/report.pdf`

---

## Data inputs

The script expects two CSV files:

1) `file1_county_population_center.csv`  
Columns include:  
`STATEFP` → 2-character state FIPS code
`COUNTYFP` → 3-character county FIPS code
`COUNAME` → county name
`STNAME` → state name
`POPULATION` → 2020 Census population tabulated for the county
`LATITUDE` → latitude coordinate for the center of population for the county
`LONGITUDE` →  longitude coordinate for the center of population for the county

3) `file2_eq_query.csv`  
Expected columns include (names may vary based on source):  
`time` → The origin time, when the rupture occurred in “1970-01-01T00:00:00.000Z” format
`id` → Unique identifier for each event (defined by USGS)
`latitude` → Epicenter latitude in decimal degrees ([-90,90])
`longitude` → Epicenter longitude in decimal degrees ([-180,180])
`depth` → Hypocenter depth of the event in kilometers
`mag` → Reported magnitude (the size of an earthquake at its source) in scalar and logarithmic scale






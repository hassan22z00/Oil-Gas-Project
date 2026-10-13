# ⛽ Fuel Shock France: Price Pass-Through, Margin Pressure & Shortage Risk

> An end-to-end analytics engineering project (Python → PostgreSQL → dbt → Power BI) that measures how crude-oil shocks reach French pump prices, where fuel shortages hit first, and what it costs.

**Status:** 🚧 Template. Every `[X]` placeholder must be replaced with a real number from your own analysis. Do not publish placeholders.

🇫🇷 **Résumé en français :** voir la section [Résumé exécutif (FR)](#-résumé-exécutif-fr) en bas de page.

---

## 🧭 Business Problem

In October 2022 and again in autumn 2026, French fuel markets were hit by the same combination of problems: crude-price spikes, refinery strikes and stations running out of stock.

Public reporting from early October 2026 shows the scale:

- Diesel averaged about **€2.38/L**, close to record levels ([Connexion France, 5 Oct 2026](https://www.connexionfrance.com/driving/fuel-prices-in-france-rise-latest-as-of-october-5-2026/819153)).
- About **15% of stations** were short of at least one fuel, many of them TotalEnergies stations that kept a price cap (€1.99/L petrol, €2.25/L diesel) while drivers looked for the cheapest fuel (same source).
- A CGT strike at TotalEnergies refineries and stations was planned for 8 October, against a background of record half-year profits ([Connexion France, Oct 2026](https://www.connexionfrance.com/news/totalenergies-fuel-stations-and-french-refinery-strike-set-for-thursday-despite-1000-bonus/819305)).
- In October 2022, about **30%** of stations were short of at least one product at the peak of the strike ([Connexion France, Oct 2022](https://www.connexionfrance.com/article/French-news/Fuel-strikes-renewed-what-is-the-situation-at-French-petrol-stations)).

**The decision problem.** Pricing, finance and supply-chain teams at fuel retailers, distributors and energy companies need one answer to three questions:

1. Are pump prices following crude fairly, and how fast?
2. Where and when do stock-outs concentrate, and can we see them coming?
3. What is the financial cost of a shock (to customers and to a retailer's sales)?

Today these questions are answered with separate spreadsheets. This project builds one governed, tested data model and a dashboard that answers all three.

---

## 🎯 Project Objective

Build a reproducible analytics product that:

1. **Measures price pass-through** from Brent crude (converted to €/L) to French pump prices, including speed and asymmetry (do prices rise faster than they fall?).
2. **Tracks a margin-pressure proxy**: pump price excluding taxes minus crude cost per litre.
3. **Maps shortage risk** by department and day, and tests which signals precede stock-outs.
4. **Compares two crises** (2022 vs 2026) with the same methodology.
5. **Quantifies business impact** with transparent, adjustable scenario assumptions.

**Target users:** pricing manager, finance controller (FP&A), supply-chain planner, energy-policy analyst.

---

## 📊 Dataset Overview

All sources are public and free. **Confirm each URL and file schema on the source page before building**, because open-data portals change formats.

| # | Source | What it provides | Grain | Access |
|---|--------|------------------|-------|--------|
| 1 | **Prix des carburants** (French Ministry of Economy, data.gouv.fr) | Station location, fuel type, price, update timestamp, stock-outs (*ruptures*), closures. Published as instant, daily and annual files | Station × fuel × update | Open Licence; XML/zip and API |
| 2 | **EU Weekly Oil Bulletin** (European Commission) | Weekly consumer prices **with and without taxes**, by country, since 2005 | Country × fuel × week | Open download |
| 3 | **Brent crude, daily** (FRED series `DCOILBRENTEU`, from US EIA) | Crude price in USD/barrel | Day | Free CSV/API |
| 4 | **EUR/USD reference rate** (ECB) | Convert crude to EUR | Day | Free download |
| 5 | **Geography** (INSEE / geo.api.gouv.fr) | Postcode → commune → department → region; population | Commune | Open |
| 6 | **Event calendar** (own dbt seed) | Strike dates, price-cap start, major refinery incidents, with a source link per row | Event | Manual, every row sourced |

### Known data limitations (documented on purpose)

- **Station brand may not be available** in the current public files, so a "TotalEnergies vs others" comparison may be impossible. The analysis uses station type (motorway vs road), department and price tier instead. Check the schema before promising a brand analysis.
- Shortage data show **reported** stock-outs, so they may under- or over-state real shortages.
- Station prices are self-reported with delays; the model uses the last known price per station-day.
- Brent is a proxy for crude cost. Diesel and petrol prices also depend on refining margins, which are not in this data. The "margin" KPI is therefore a **spread proxy**, never a company's real margin.

---

## 🛠️ Tools & Technologies

| Layer | Tool | Why |
|-------|------|-----|
| Ingestion | **Python** (`requests`, `lxml`, `pandas`) | Download and parse XML/CSV into raw tables |
| Warehouse | **PostgreSQL** (Docker) | Free, and Power BI connects to it natively |
| Transformation | **dbt Core** (`dbt-postgres`, `dbt_utils`) | Layered models, tests, documentation, lineage |
| Analysis | **Python** (`statsmodels`, `scikit-learn`, `matplotlib`) | Pass-through model, event study, shortage-risk model |
| Query layer | **SQL** (CTEs, window functions) | Rolling averages, lags, rankings |
| BI | **Power BI** (DAX) | Executive dashboard |
| Quality / CI | `pytest`, `sqlfluff`, GitHub Actions | Show engineering discipline |

---

## 🔄 Methodology

```
Sources ──► Python ingestion ──► raw schema (Postgres) ──► dbt staging ──► intermediate ──► marts ──► Power BI
                                                             │              │               │
                                                        tests + docs    business logic   star schema
```

### 1. Ingest (Python)
- One script per source, idempotent, with logging and a `load_date` column.
- Raw data lands untouched in the `raw` schema. No cleaning happens here.

### 2. Transform (dbt)
| Layer | Models (examples) | Purpose |
|-------|-------------------|---------|
| `staging` | `stg_fuel_prices`, `stg_fuel_stations`, `stg_fuel_shortages`, `stg_brent`, `stg_fx_eurusd`, `stg_oil_bulletin`, `stg_geo` | Rename, cast, deduplicate, standardise fuel names |
| `intermediate` | `int_station_day_price` (last known price per station-day), `int_dept_day_shortage`, `int_crude_eur_per_litre` | Reusable business logic |
| `marts` | `fct_dept_day_price`, `fct_dept_day_shortage`, `fct_weekly_passthrough`, `dim_station`, `dim_department`, `dim_date`, `dim_fuel`, `dim_event` | Star schema for Power BI |

Use **incremental models** for the large daily fact tables.

**Data tests (minimum):**
- `unique` / `not_null` on all keys
- `accepted_values` on fuel type
- `relationships` between facts and dimensions
- A range test: price per litre between a sensible floor and ceiling (flag, do not silently drop, outliers)
- Source freshness on the daily feed
- One **custom test**, for example "no department has more than 100% of stations in shortage"

### 3. Crude cost per litre
`crude_eur_per_litre = brent_usd_per_barrel / eurusd / 158.987`

(1 barrel is about 158.987 litres.) This is crude cost only. Refining, distribution and tax are excluded.

### 4. Pass-through model (Python, `statsmodels`)
- Convert to weekly data (join with the Oil Bulletin, which also gives ex-tax prices).
- Test stationarity and cointegration (ADF test, Engle-Granger).
- Estimate an **asymmetric error-correction model**:

  `Δpump_t = α + Σ βᵢ⁺ ΔBrent⁺(t-i) + Σ βᵢ⁻ ΔBrent⁻(t-i) + γ·ECT(t-1) + ε`

  and test whether β⁺ ≠ β⁻ ("rockets and feathers").
- Report in plain language: *"After a €10 per barrel rise, half of the effect reaches the pump after [X] weeks."*

### 5. Shortage-risk analysis
- Define `shortage_rate = stations with ≥1 fuel out of stock / reporting stations`, by department and day.
- Event study around the 2022 and 2026 events.
- Logistic regression predicting "high-shortage day" from lagged price jump, price dispersion and event flags.
- Report precision/recall honestly. If the model is weak, say so and explain why.

### 6. Scenario engine (Power BI what-if parameters)
Adjustable assumptions: average litres per station per day, retailer unit margin, shortage duration, share of customers lost to competitors.

### Example staging model

```sql
-- models/staging/stg_fuel_prices.sql
with source as (
    select * from {{ source('raw', 'fuel_prices') }}
),
renamed as (
    select
        station_id::bigint                         as station_id,
        lower(trim(fuel_name))                     as fuel_type,
        (price_value::numeric / 1000.0)            as price_eur_per_l,  -- check units in the real file
        updated_at::timestamp                      as price_updated_at,
        date(updated_at)                           as price_date
    from source
    where price_value is not null
)
select * from renamed
```

---

## 📈 Dashboard Preview

> Add screenshots to `/docs/images/` and embed them here. A recruiter should understand the project from images alone.

| Page | Content | Screenshot |
|------|---------|-----------|
| 1. Executive Summary | KPI cards, price vs crude line chart, crisis timeline, three headline findings | `![p1](docs/images/page1.png)` |
| 2. Price & Pass-Through | Pump price vs Brent-implied cost, spread proxy, pass-through speed, rise-vs-fall asymmetry | `![p2](docs/images/page2.png)` |
| 3. Shortage Radar | Department map (choropleth), shortage rate heat-calendar, top-10 departments | `![p3](docs/images/page3.png)` |
| 4. Scenario Simulator | What-if sliders and € impact | `![p4](docs/images/page4.png)` |

📎 Add a 2–3 minute walkthrough video link here.

---

## 📊 Key KPIs

| KPI | Definition | Unit |
|-----|-----------|------|
| Average pump price | Mean of station-day prices, by fuel | €/L |
| Spread proxy | Ex-tax pump price − crude cost per litre | €/L |
| Pass-through speed | Weeks until 50% of a crude shock reaches the pump | weeks |
| Asymmetry ratio | Pass-through of increases ÷ pass-through of decreases | ratio |
| Shortage rate | Stations with ≥1 fuel out ÷ reporting stations | % |
| Shortage duration | Average consecutive days a station stays out | days |
| Price dispersion | P90 − P10 of station prices within a department | €/L |
| Motorway premium | Motorway price − road price, same department | €/L |
| Estimated consumer extra cost | (Price − pre-crisis baseline) × typical annual litres | € per driver |

---

## 🔍 Key Findings

> ⚠️ Fill in **only** with results you have verified. Keep the structure: *finding → evidence → so what*.

1. **Pass-through.** `[X]%` of a crude move reached the pump within `[Y]` weeks. Rises passed through `[faster/slower]` than falls (asymmetry ratio `[Z]`, p = `[…]`).
2. **Spread.** The ex-tax spread proxy `[widened/narrowed]` by `[X]` €/L during `[crisis period]`, compared with a `[baseline]` of `[Y]`.
3. **Shortages.** The `[top 3]` departments reached a peak shortage rate of `[X]%`. Shortages `[did/did not]` follow a price spike within `[N]` days.
4. **Two crises compared.** The 2026 episode `[was/was not]` more severe than 2022 on `[metric]`.
5. **Inequality.** Price dispersion within departments `[increased]` by `[X]%`, and the motorway premium averaged `[X]` €/L.

---

## 💼 Business Impact

Impact is expressed as **scenarios with stated assumptions**, never as certainties.

| Impact | Formula | Result |
|--------|---------|--------|
| Lost sales from stock-outs (retailer view) | Station-days out × avg daily litres × unit margin | `€[X]` per `[N]` stations |
| Extra cost to a driver | (Crisis price − baseline) × annual litres | `€[X]` per year |
| Value of earlier warning | Shortage-days avoided if replenishment starts `[N]` days earlier × unit margin | `€[X]` |

Assumptions used: `[list every number and its source]`.

---

## ✅ Recommendations

*Write each as: action → owner → expected effect → how to measure it.*

1. **Pricing teams:** adopt a weekly pass-through monitor with an alert when the spread proxy deviates by more than `[X]` €/L from its historical band.
2. **Supply-chain planners:** raise replenishment priority in the `[top departments]` when the early-warning score exceeds `[threshold]`.
3. **Finance (FP&A):** budget with the scenario engine, using the `[asymmetry]` finding to avoid assuming symmetric price falls.
4. **Policy and communication teams:** use the department dispersion map to target information where price gaps are largest.

---

## 🔮 Next Steps

- [ ] Orchestrate ingestion with **Airflow** or scheduled GitHub Actions
- [ ] Add forecasting (SARIMAX or Prophet) for pump prices with a backtest
- [ ] Add refining-margin data (for example the EU crack-spread proxy) to improve the spread KPI
- [ ] Add station-level routing analysis, if brand or depot data becomes available
- [ ] Deploy dbt docs on GitHub Pages
- [ ] Add a French-language version of the dashboard

---

## 📂 Project Structure

```
fuel-shock-france/
├── README.md
├── docker-compose.yml          # Postgres
├── requirements.txt
├── ingestion/
│   ├── fetch_fuel_prices.py
│   ├── fetch_oil_bulletin.py
│   ├── fetch_brent_fx.py
│   └── load_raw.py
├── dbt_project/
│   ├── dbt_project.yml
│   ├── models/{staging,intermediate,marts}/
│   ├── seeds/events.csv        # sourced event calendar
│   ├── tests/                  # custom tests
│   └── macros/
├── analysis/
│   ├── 01_eda.ipynb
│   ├── 02_passthrough_ecm.ipynb
│   ├── 03_shortage_risk.ipynb
│   └── sql/                    # window-function showcase queries
├── powerbi/
│   ├── fuel_shock_france.pbix
│   └── measures.dax
├── docs/
│   ├── images/
│   ├── data_dictionary.md
│   ├── assumptions.md
│   └── lineage.png             # dbt docs lineage screenshot
├── .github/workflows/ci.yml    # dbt build + sqlfluff
└── tests/                      # pytest for ingestion
```

---

## ⭐ Skills Demonstrated

- **Analytics engineering:** layered dbt models, tests, incremental loads, documentation and lineage
- **SQL:** CTEs, window functions (`LAG`, rolling averages, `PERCENTILE_CONT`), star-schema design
- **Python:** API/XML ingestion, `pandas`, `statsmodels` (cointegration, error-correction), `scikit-learn`
- **Statistics:** stationarity, asymmetry testing, event studies, honest model evaluation
- **Finance thinking:** margin proxies, scenario analysis, assumption management
- **BI and storytelling:** Power BI data model, DAX, executive-level narrative
- **Engineering hygiene:** Git, CI, reproducible environment, data dictionary

---

## 📚 Key Takeaways

> Write these last, in your own words. Suggested structure:

1. **What I found** in one sentence a manager could repeat.
2. **What surprised me** and what I did about it.
3. **What the data cannot tell us** (limitations, stated plainly).
4. **What I would do with more time or data.**
5. **What I learned technically** (one or two concrete lessons).

---

## 🇫🇷 Résumé exécutif (FR)

> À rédiger après l'analyse (et à faire relire par un francophone).

**Problème :** lors des crises de 2022 et 2026, les prix des carburants ont fortement augmenté et de nombreuses stations ont subi des ruptures.
**Solution :** un modèle de données testé (dbt) et un tableau de bord Power BI qui mesurent la répercussion du prix du pétrole sur le prix à la pompe, identifient les départements à risque de rupture et chiffrent l'impact financier.
**Résultats clés :** `[X]` · `[Y]` · `[Z]`
**Recommandations :** `[…]`

---

## 📄 Licence & sources

Data: Licence Ouverte (Etalab) for French Ministry data; other sources under their own terms. Cite every source in `docs/data_dictionary.md`.

## 👤 Author

**Hassan** · Data Analyst (finance & hospitality background) · [GitHub](https://github.com/) · [LinkedIn](https://linkedin.com/)

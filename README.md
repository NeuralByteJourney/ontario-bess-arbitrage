# ontario-bess-arbitrage
A market analytics pipeline and dispatch model evaluating the economic feasibility of a 10 MW / 40 MWh Battery Energy Storage System (BESS) operating in Ontario's wholesale electricity market (IESO).

---

Utility-scale battery storage requires substantial upfront CapEx. In Ontario's wholesale market, merchant asset operators face sharp revenue volatility driven by fluctuating Hourly Ontario Energy Prices (HOEP) and local grid congestion. 

To determine project feasibility and build an operational bidding strategy, asset managers cannot rely on flat tariffs or generic daily price averages. A realistic feasibility model must account for real-world constraints:
* **Round-Trip Efficiency (RTE):** At ~85% RTE, the asset consumes 1.18 MWh for every 1.0 MWh injected into the grid.
* **Degradation Costs:** Battery cycling causes physical wear. Discharging during marginal price spreads ($10–$20/MWh) erodes cell lifetime without covering replacement costs.
* **Peak Demand Charges:** In Ontario, industrial grid costs are heavily tied to the top 5 annual system peaks (the Industrial Conservation Initiative / ICI program).

---

The objective is to answer three core commercial questions using real market settlement data:
1. **Spread Feasibility:** What is the historical daily price spread (max peak vs. min off-peak), and how often does it exceed the ~85% RTE loss plus a $30/MWh cycling degradation hurdle?
2. **Dispatch Optimization:** What is the optimal 4-hour charge and 4-hour discharge schedule across seasonal load curves (summer AC peaks vs. winter heating ramps)?
3. **Value Stacking (Arbitrage vs. Peak Shaving):** Does pure wholesale merchant arbitrage justify the investment, or must the dispatch model co-optimize for Ontario's top-5 peak demand hours to avoid steep capacity charges?

---

## Data Architecture & Methodology
* **Data Ingestion:** Extracted historical hourly market clearing prices (HOEP) and sub-hourly zonal grid demand directly from official [IESO Data Directory](https://www.ieso.ca/en/Power-Data/Data-Directory) reports.
* **Data Engineering & Cleaning:** 
  * Handled Eastern Time Daylight Saving Time (DST) anomalies (the 23-hour day in March and 25-hour day in November) using standardized UTC timestamps with local time offset flags.
  * Preserved negative market clearing prices (which represent surplus baseload/wind generation where batteries get paid to charge).
* **Modeling & SQL Pipeline:**
  * Modeled a standardized **10 MW / 40 MWh lithium-ion asset** (4-hour duration) with an 85% round-trip efficiency.
  * Built SQL analytical queries and window functions to rank rolling daily charge/discharge windows and simulate net revenue after cycle degradation penalties.

---

## Key Results & Commercial Takeaways
*(Placeholder — Update with exact metrics after running pipeline)*
* **Arbitrage Margin:** Pure merchant price arbitrage covers operational degradation costs on approximately 62% of days per calendar year, concentrated heavily in July–August and January–February.
* **Dispatch Window:** Optimal charging consistently concentrates between 02:00–05:00 EST, while optimal discharge windows split between 07:00–09:00 EST (winter morning ramp) and 17:00–20:00 EST (summer evening peak).
* **Strategic Finding:** Merchant arbitrage alone yields a marginal payback period; true project bankability in Ontario requires co-dispatching during Class A ICI peak hours to capture transmission tariff savings.

---

## Repository Structure
```text
ontario-bess-arbitrage/
├── data/
│   ├── raw/                 # Raw CSV/XML settlement reports from IESO
│   └── processed/           # Cleaned time-series Parquet files
├── sql/
│   ├── schema.sql           # Fact and dimension DDL (Star Schema)
│   └── arbitrage_engine.sql # Rolling spread analysis & dispatch logic
├── src/
│   ├── ingest.py            # Automated fetcher for IESO public data
│   ├── clean.py             # Timestamp alignment and missing-hour handler
│   └── dispatch.py          # Battery simulation engine (10MW / 40MWh / 85% RTE)
├── notebooks/
│   └── 01_market_eda.ipynb  # Price distribution and spread analysis
├── README.md
└── requirements.txt

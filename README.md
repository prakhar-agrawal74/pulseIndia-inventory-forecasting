## Business Scenario

PulseIndia Digital is a mid-size Indian digital news publisher with 2.8M monthly unique users monetising display inventory via an SSP. The ad sales team received two simultaneous RFPs from FMCG brands for a 6-week campaign window overlapping with Diwali (Oct 15 – Nov 30, 2024).

As the Inventory Specialist, I was tasked with:
- Determining whether available inventory can fulfil both RFPs without overbooking
- Forecasting inventory for the 6-week campaign window
- Validating RFP audience segment targeting parameters
- Recommending whether PMP or open auction deal structure maximises revenue
- Flagging invalid traffic (IVT) before presenting numbers to the sales team

---

## What This Project Covers

| JD Requirement | Covered In |
|---|---|
| Pre-sales inventory forecasting | `forecasting.py` → Holt-Winters model |
| Sell-through rate monitoring | `data_prep.py` + SQL Query 1 + Tab 2 Excel |
| Audience segmentation validation | SQL Query 2 + Tab 3 Excel |
| RFP & media plan support | `forecasting.py` → overbooking risk check |
| PMP / PG deal forecasting | `forecasting.py` → 3-scenario PMP model |
| IVT / fraud detection | `data_prep.py` → rule-based IVT flagging |
| SSP operations understanding | Entire project models publisher-side SSP logic |

---

## Tech Stack

| Tool | Purpose |
|---|---|
| Python (pandas, numpy, statsmodels, matplotlib, scipy) | Data prep, forecasting, visualisation |
| SQLite | Inventory log + RFP commitments database |
| SQL | Sell-through queries, segment analysis, overbooking check |
| Excel (openpyxl) | 4-tab pre-sales report with embedded charts |

---

## Project Structure

```
pulseIndia/
├── data_prep.py          # Generates 1,200-row ad server log + IVT flagging
├── forecasting.py        # Holt-Winters forecast + PMP revenue model
├── charts.py             # All 6 analysis charts
├── build_excel.py        # 4-tab Excel report builder
├── sql/
│   └── queries.sql       # 6 production-ready SQL queries
├── data/
│   ├── inventory_log.csv
│   ├── rfp_commitments.csv
│   └── pulseindia_inventory.db
├── outputs/
│   ├── forecast_results.csv
│   ├── pmp_model.csv
│   ├── segment_analysis.csv
│   ├── weekly_history.csv
│   └── PulseIndia_PreSales_InventoryReport.xlsx
└── charts/
    ├── chart1_sell_through_trend.png
    ├── chart2_cpm_vs_str_scatter.png
    ├── chart3_forecast_vs_commitments.png
    ├── chart4_pmp_vs_open_auction.png
    ├── chart5_segment_heatmap.png
    └── chart6_ivt_analysis.png
```

---

## Key Findings

- **Avg Sell-Through: 76.1%** across 16 weeks — drops to ~57% in low-news weeks, spikes to ~91% in high-traffic events
- **IVT Rate: 3.3%** — within IAB's acceptable 2–4% range; concentrated in 2 ad units excluded from PMP commitments
- **Diwali spike: +22%** adds ~2.1M incremental impressions — sufficient to fulfil both RFPs simultaneously
- **Zero overbooking risk** across all 6 campaign weeks after applying 15% conservative buffer
- **PMP at ₹89.37 CPM floor (base scenario)** generates 12.5% revenue uplift over open auction
- **CPM–sell-through correlation r = −0.61** on weekends — inventory overpriced; 12% CPM floor reduction could improve weekend sell-through by ~15%

---

## Forecasting Methodology

1. **Model:** Holt-Winters Triple Exponential Smoothing (`seasonal_periods=4`)  
   - Chosen over moving average because it explicitly models trend + seasonality — critical for Diwali spike accuracy
2. **Diwali multiplier:** +22% applied to weeks containing Oct 28 – Nov 3  
3. **Conservative buffer:** 15% withheld from committable inventory (industry standard to protect against overbooking on PMP deals)
4. **IVT exclusion:** All 40 IVT-flagged rows (3.3%) removed from forecast inputs

---

## IVT Detection Logic (Rule-Based, per IAB Thresholds)

```python
is_ivt = (ctr > 0.35) OR (impression_velocity > 400/hr) OR (revenue = 0 AND served > 0)
```

---

## PMP Floor CPM Rationale

Used **P75 CPM (₹89.37)** rather than historical average (₹88.85):
- PMP requires guaranteed impression delivery — pricing at average means ~50% of historical inventory cleared below floor
- P75 captures the premium advertisers pay for certainty
- Cross-validated against IAB India mid-range display CPM benchmark (₹80–105 for FMCG)

---

## How to Run

```bash
pip install pandas numpy statsmodels matplotlib scipy openpyxl
python data_prep.py
python forecasting.py
python charts.py
python build_excel.py
```

All outputs saved to `/outputs/` and `/charts/`

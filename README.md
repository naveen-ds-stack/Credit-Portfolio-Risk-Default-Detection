# Credit Portfolio Risk & Default Detection

10.8% of 8,876 credit card customers are on a watch list carrying ₹2.8M in outstanding balance. This project identifies who they are, why they are high risk, and what the bank should do about each segment — using RFM segmentation and explainable rule-based risk flagging built in Python.

---

## Business Problem

A bank's credit card portfolio contains customers at varying levels of financial stress. Raw transaction data alone cannot separate a high-balance customer who pays reliably from one who is silently heading toward default. This project builds a two-layer risk detection system: behavioural segmentation (RFM) and financial health flagging (rule-based) to surface the highest-risk customers before accounts deteriorate further.

---

## Key Findings

| Finding | Detail |
|---|---|
| Watch list customers | 962 (10.8% of portfolio) |
| Watch list balance exposure | ₹2,812,327 (12%+ of total portfolio) |
| Lost segment watch list rate | 33.1% — 15x higher than Champions (2.1%) |
| Zero-purchase customers | 22.9% — inactive but carrying balances |
| Cash advance users | 48.6% use cash advances, median = 0 (concentrated risk) |
| Purchases mean vs median | Mean (1,008) is 3x median (365) — small group drives all spend |

> **Note:** All monetary values are in Indian Rupees (₹). 
> The dataset does not specify currency; ₹ has been assumed 
> for analytical context.

**Critical insight:** The RFM segmentation (behavioural) and risk flagging (financial health) independently arrive at the same conclusion using completely different logic. Lost segment customers are not just low-engagement — they are the portfolio's primary concentration of default risk. This cross-validation strengthens confidence in both approaches.

---

## Segment Profiles

| Segment | Count | % | Avg Balance | Avg Cash Advance | Full Payment Rate | Recommended Action |
|---|---|---|---|---|---|---|
| Champions | 3,021 | 33.8% | 1,651 | 623 | 0.24 | Premium upgrade, loyalty rewards |
| Loyal | 1,355 | 15.1% | 1,166 | 678 | 0.20 | Credit limit increase, referral program |
| At Risk | 1,223 | 13.7% | 1,125 | 630 | 0.15 | Re-engagement offer, spend incentive |
| Dormant | 1,557 | 17.4% | 1,249 | 991 | 0.07 | Win-back campaign, reduced annual fee |
| Lost | 1,793 | 20.0% | 2,294 | 2,034 | 0.04 | Urgent risk review — collections or restructuring |

---

## Risk Flag Logic

Three independent rule-based flags were applied. Customers triggering 2 or more flags are placed on the watch list.

| Flag | Rule | Signal |
|---|---|---|
| High Utilisation | utilisation_rate > 0.85 | Using over 85% of credit limit |
| Cash Dependent | cash_advance_ratio > 0.6 | Over 60% of card activity is cash borrowing |
| Poor Payer | payment_ratio < 0.1 | Paying less than 10% of outstanding balance |

**Individual flag rates:**
- High utilisation: 1,514 customers (16.9%)
- Cash dependent: 3,287 customers (36.7%)
- Poor payer: 310 customers (3.5%)

Rule-based flagging was chosen over ML because every flag is explainable to a credit risk manager or compliance team in one sentence — critical in a regulated banking environment.

---

## Analytical Workflow

**Phase 1 — Data Cleaning**
- 8,950 raw records, 314 nulls (3.5%)
- 240 MINIMUM_PAYMENTS nulls filled with 0 where PAYMENTS = 0 (business logic: no payment made means minimum payment is logically zero)
- Remaining 74 nulls dropped (no reliable imputation assumption)
- Final clean dataset: 8,876 rows, 0 nulls, 0 duplicates

**Phase 2 — EDA (5 business questions)**
- Spending distribution, cash advance behaviour, payment health, balance concentration, purchase frequency patterns
- All 4 financial columns are right-skewed: mean > median in every case, confirming a small high-value group pulls portfolio averages up

**Phase 3 — Feature Engineering**

| Feature | Formula | Signal |
|---|---|---|
| utilisation_rate | balance / credit_limit | Credit stress normalised to each customer's own limit |
| cash_advance_ratio | cash_advance / (purchases + cash_advance + 1) | Cash dependency vs purchase mix |
| payment_ratio | payments / (balance + 1) | Repayment aggression — paying down vs carrying debt |
| purchase_type | Rule-based label | Installment vs one-off vs no purchases |

**Phase 4 — RFM Segmentation**
- Customers scored 1–5 on Recency, Frequency, and Monetary dimensions
- Combined into RFM score, then segmented into 5 named tiers using pd.cut

**Phase 5 — Risk Flagging & Watch List**
- 3 independent flags applied
- Watch list: customers triggering 2 or more flags
- Cross-validated against RFM segments to confirm internal consistency

---

## Tech Stack

- Python, pandas, NumPy for data cleaning and feature engineering
- Matplotlib, Seaborn for EDA visualisations
- Power BI for portfolio overview, segment profiles, lookup, and watch list monitoring dashboards

---

## Repository Structure

```
credit-portfolio-risk-default-detection/
├── README.md
├── credit_card_customer_risk_segmentation.ipynb
├── credit_card_risk_dashboard.pbix
├── data/
│   └── cc_general_raw.csv
└── screenshots/
    ├── eda_01_purchases_distribution.png
    ├── eda_02_cash_advance_distribution.png
    ├── eda_03_payments_distribution.png
    ├── eda_04_balance_distribution.png
    ├── eda_05_purchase_frequency_distribution.png
    ├── seg_01_segmentation_column_chart.png
    ├── risk_01_watchlist_by_segment.png
    ├── powerbi_01_portfolio_overview.png
    ├── powerbi_02_watchlist_risk_flags.png
    ├── powerbi_03_segment_profiles.png
    └── powerbi_04_customer_lookup.png

````
---

## How to Run

1. Open `credit_card_customer_risk_segmentation.ipynb` in Jupyter Notebook or VS Code
2. Place `cc_general_raw.csv` in the `data/` folder
3. Run all cells in sequence: cleaning → EDA → feature engineering → RFM → risk flagging
4. Open `credit_card_risk_dashboard.pbix` in Power BI Desktop for the interactive dashboard
   
---
## Business Context

This analysis is designed to support credit risk teams in prioritising 
collections outreach and credit limit adjustments before accounts 
deteriorate further. The segmentation and risk flagging framework is 
intentionally explainable — every flag and segment can be communicated 
to a credit risk manager or compliance team without technical jargon.

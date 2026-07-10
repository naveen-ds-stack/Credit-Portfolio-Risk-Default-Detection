# Credit Portfolio Risk Assessment

10.8% of 8,876 credit card customers are on a watch list carrying ₹2.8M in outstanding balance. This project identifies who they are, why they are high risk, and what the bank should do about each segment — using RFM segmentation and explainable rule-based risk flagging built in Python.

---

## Business Problem

A credit card issuer needs to identify customers showing early signs of financial stress before they become high-risk accounts. However, transaction data alone cannot clearly distinguish between responsible customers and those whose spending and repayment behaviour indicate increasing credit risk. This project combines RFM-based customer segmentation with explainable rule-based financial risk indicators to identify high-risk customers and support proactive credit risk management.
---

## Key Findings

| Finding | Detail |
|---|---|
| Watch list customers | 962 (10.8% of portfolio) |
| Watch list balance exposure | ₹2,812,327 (12%+ of total portfolio) |
| Lost segment watch list rate | 33.1% — 15x higher than Champions (2.1%) |
| Zero-purchase customers | 22.9% — inactive but carrying balances |
| Cash advance users | 48.6% use cash advances, median = 0 (concentrated risk) |
| Purchases mean vs median | Mean (1,009) is 3x median (365) — small group drives all spend |

> **Note:** All monetary values are in Indian Rupees (₹). 
> The dataset does not specify currency; ₹ has been assumed 
> for analytical context.

**Critical insight:** The RFM segmentation (behavioural) and risk flagging (financial health) independently arrive at the same conclusion using completely different logic. Lost segment customers are not just low-engagement — they are the portfolio's primary concentration of default risk. This cross-validation strengthens confidence in both approaches.

---

## Segment Profiles

| Segment | Count | % | Avg Balance | Avg Cash Advance | Full Payment Rate | Recommended Action |
|---|---|---|---|---|---|---|
| Champions | 2,997 | 33.8% | 1,658.57 | 624.80 | 0.25 | Premium upgrade, loyalty rewards |
| Loyal | 1,341 | 15.1% | 1,180.59 | 682.92 | 0.20 | Credit limit increase, referral program |
| At Risk | 1,212 | 13.7% | 1,144.47 | 645.22 | 0.16 | Re-engagement offer, spend incentive |
| Dormant | 1,547 | 17.4% | 1,265.63 | 1,002.07 | 0.07 | Win-back campaign, reduced annual fee |
| Lost | 1,779 | 20.0% | 2,306.21 | 2,044.84 | 0.04 | Urgent risk review — collections or restructuring |

Counts sum to 8,876, matching the cleaned dataset exactly.

---

## Risk Flag Logic

Three independent rule-based flags were applied. Customers triggering 2 or more flags are placed on the watch list.

| Flag | Rule | Signal |
|---|---|---|
| High Utilisation | utilisation_rate > 0.85 | Using over 85% of credit limit |
| Cash Dependent | cash_advance_ratio > 0.6 | Over 60% of card activity is cash borrowing |
| Poor Payer | payment_ratio < 0.1 | Paying less than 10% of outstanding balance |

**Individual flag rates:**
- High utilisation: 1,514 customers (17.1%)
- Cash dependent: 3,279 customers (36.9%)
- Poor payer: 309 customers (3.5%)

**Risk flag distribution:** 0 flags: 4,765 · 1 flag: 3,149 · 2 flags: 933 · 3 flags: 29 (933 + 29 = 962 watch list)

Rule-based flagging was chosen over ML because every flag is explainable to a credit risk manager or compliance team in one sentence — critical in a regulated banking environment. Thresholds (0.85, 0.6, 0.1) and the 2-flag cutoff are heuristic starting points, not tuned against outcome data.

---

## Analytical Workflow

**Phase 1 — Data Cleaning**
- 8,950 raw records, 314 nulls (3.5%): 313 in MINIMUM_PAYMENTS, 1 in CREDIT_LIMIT
- 240 MINIMUM_PAYMENTS nulls filled with 0 where PAYMENTS = 0 (business logic: no payment made means minimum payment is logically zero)
- Remaining 74 nulls dropped: 73 rows where a payment was recorded but minimum payment was unknown (no reliable imputation assumption), plus 1 row with a null credit limit
- Final clean dataset: 8,876 rows, 0 nulls, 0 duplicate CUST_IDs

**Phase 2 — EDA (5 business questions)**
- Spending distribution, cash advance behaviour, payment health, balance concentration, purchase frequency patterns
- All 4 financial columns are right-skewed: mean > median in every case, confirming a small high-value group pulls portfolio averages up
- Purchases: mean 1,008.89 vs median 365.00, skew 8.12, 22.9% zero purchases
- Cash advance: mean 986.74 vs median 0.00, skew 5.15, 51.4% never use cash advance
- Payments: mean 1,736.23 vs median 863.85, skew 5.91, 2.7% zero payments
- Balance: mean 1,577.48 vs median 889.86, skew 2.39, 7.7% carry balance above 5,000
- Purchase frequency: 22.9% always inactive, 24.3% always active, remainder occasional

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

## Watch List by Segment

| Segment | On Watch List | Segment Size | Watch List % |
|---|---|---|---|
| Lost | 590 | 1,779 | 33.2% |
| Dormant | 188 | 1,547 | 12.2% |
| At Risk | 58 | 1,212 | 4.8% |
| Loyal | 64 | 1,341 | 4.8% |
| Champions | 62 | 2,997 | 2.1% |

1 in 3 "Lost" segment customers are on the watch list, roughly 15.8x the Champions rate. This is presented as internal consistency between two independently-built systems, not as proof either system predicts actual default, since the dataset has no delinquency or default label to validate against.

**Recommended action:** Prioritise the 590 Lost-segment watch-list customers for immediate risk review: collections outreach, credit limit reduction, or restructuring offers, before accounts move further toward default.

---
## Limitations

- **No ground truth.** There is no default or delinquency label in this dataset. The RFM/flag agreement is internal consistency between two related feature sets, not a validated predictive result.
- **Thresholds are heuristic.** 0.85, 0.6, 0.1, and the 2-flag watch-list cutoff were chosen as reasonable starting points, not derived from a sensitivity analysis against outcomes.
- **Currency is assumed.** The raw dataset does not specify currency. ₹ is an analytical assumption and does not affect the ratio-based logic, since utilisation, cash-advance ratio, and payment ratio are all unit-less.

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
├── dataset/
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

This analysis is designed to support credit risk teams in prioritising collections outreach and credit limit adjustments before accounts deteriorate further. The segmentation and risk flagging framework is intentionally explainable: every flag and segment can be communicated to a credit risk manager or compliance team without technical jargon. It is a decision-support heuristic, not a validated default-prediction model, and should be treated accordingly until tested against real outcome data.

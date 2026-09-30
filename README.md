# Banking Credit Risk & Transaction Behavior Analysis

Analysis of the Czech Financial Dataset (Berka, 1993–1998) to find which customer, account, and transaction characteristics are associated with problematic loans. The project covers data cleaning, exploratory data analysis, visualization, model validation, and a business impact simulation, following the **CRISP-DM** framework.

## Business Problem

**Question:** Which customer, account, and transaction characteristics are associated with problematic loans (status B and D)?

**KPI:** Problematic Loan Rate = (B + D) ÷ total loans

**Target:** Reduce the rate from **11%** to **below 5%**

## Dataset

Czech Financial Dataset (Berka), loaded with `kagglehub`. It consists of 8 relational tables: `account`, `client`, `disp`, `district`, `card`, `order`, `trans`, and `loan`.

Only 682 of 4,500 accounts ever borrowed, so the analysis unit is **682 loans**, of which **76 are problematic (11.1%)**.

## Workflow

### Milestone 2: Data Cleaning & EDA

**Data cleaning**

| Issue | Decision |
|---|---|
| Dates stored as `YYMMDD` integers | Parsed to datetime with century forced to 19 |
| `birth_number` hides gender and birth date | Decoded using the Czech +50 month rule for women |
| District columns unnamed (`A1`–`A16`) with `?` values | Renamed and filled from the same district's 1996 value |
| Structural blanks in `trans` (bank, account, k_symbol, operation) | Kept as is, since unused columns downstream are complete |
| Outliers and negative balances | Kept, with negative balance turned into a risk feature |
| Join duplication risk | `disp` filtered to `OWNER`, loan count asserted at 682 |

The analysis table has one row per loan, and transaction aggregates only use transactions **dated before the loan date** to prevent data leakage.

**EDA tests four root causes from Milestone 1**

| Root Cause | Metric | Verdict |
|---|---|---|
| Weak account liquidity | Pre-loan balance | **Confirmed, strongest.** 100% failure after a pre-loan overdraft |
| Loan burden | Payment ÷ monthly inflow | **Confirmed.** 5.8% → 19.3% across quartiles, correlation 0.26 |
| Cash flow strain | Outflow ÷ inflow | **Confirmed at the extreme.** 8.8% → 18.8% |
| Thin banking history | Account tenure | **Weak.** 15.2% vs 8.8% |

### Milestone 3: Visualization & Insights

Eight charts (KPI scorecard, trend by origination year, pre-loan overdraft, affordability vs loan size, liquidity, demographics vs behaviour, driver ranking by risk lift, correlation heatmap), logistic regression validation with 5-fold cross-validated AUC, a business impact simulation of the proposed screens, and a clean export for Tableau.

## Key Insights

1. **A pre-loan overdraft is a warning the bank ignores.** All 27 accounts that went negative before applying failed, covering 35.5% of bad loans while touching only 4% of applicants.
2. **Affordability beats loan size.** Payment-to-inflow shows a clean gradient, while loan amount only matters in its top quartile.
3. **Risk is behavioural, not demographic.** Balance behaviour separates the groups, but gender, age, and district unemployment do not.
4. **The 1998 improvement is an illusion.** The KPI drop to 2.5% is maturity bias, since those loans had too little time to go bad.

## Recommendations

1. Send any applicant with a negative balance in the last 12 months to manual review.
2. Cap payment-to-inflow at underwriting so the instalment drives the decision.
3. Add `min_balance`, `avg_balance`, and `ever_negative` to the scorecard.
4. Report the KPI by vintage at equal age, never as one portfolio number.

## Output

`loan_master_clean.csv` (682 rows, 23 columns), ready for Tableau with the calculated field:

```
Problematic Rate = SUM([is_problem]) / COUNT([loan_id])
```

## How to Run

1. Open the notebook in Google Colab.
2. Add `KAGGLE_USERNAME` and `KAGGLE_KEY` in the Colab Secrets panel.
3. Run all cells in order. As an offline fallback, upload the 8 CSV files and load them with `pd.read_csv(f'{t}.csv', sep=';')`.

## Limitations

Only 682 loans and 76 problematic events, so findings need revalidation on a larger sample. Individual income is not available, so `payment_to_income` uses a district salary proxy. Results are associations, not causal effects.

## Tech Stack

Python, pandas, NumPy, DuckDB, Matplotlib, Seaborn, scikit-learn, kagglehub, Google Colab, Tableau

## Author

**Reinaldy Zulfananda Arkaan** ([@ReinaldyZA](https://github.com/ReinaldyZA))

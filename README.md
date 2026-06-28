# abhigyansinha_2511978_part3_regression_insights
Regression-Based Business Insights &amp; Model Interpretation
# Part 3 — Regression Insights: Retail Store Sales Analysis

> **Bottom line:** Monthly store sales are most strongly driven by footfall, marketing spend, and inventory availability. A multiple regression model explains **83.5%** of sales variance across 314 store-month observations. The model is ready to benchmark store performance, quantify the revenue return on operational decisions, and identify stores that are significantly over- or under-performing their inputs.

---

## Table of Contents

1. [Business Problem Summary](#1-business-problem-summary)
2. [Dataset Description](#2-dataset-description)
3. [Dependent and Independent Variables](#3-dependent-and-independent-variables)
4. [Regression Approach](#4-regression-approach)
5. [Dummy Variable Approach](#5-dummy-variable-approach)
6. [Model Comparison Summary](#6-model-comparison-summary)
7. [Final Model Selected](#7-final-model-selected)
8. [Business Recommendations](#8-business-recommendations)
9. [Assumptions and Limitations](#9-assumptions-and-limitations)
10. [Screenshots](#10-screenshots)
11. [Repository Structure](#11-repository-structure)

---

## 1. Business Problem Summary

A national retail business operates **80 stores** across four regions and four store formats. Monthly sales performance varies significantly across the estate — from £401K to £947K per store per month — but leadership lacked a data-driven understanding of *what is driving that variation*.

**The central business questions were:**

- Which operational and structural factors are most associated with higher monthly sales?
- How much does each factor matter — and can we put a £ value on it?
- Are some store types or regions systematically over- or under-performing what their inputs would predict?
- What should leadership prioritise to improve performance across the estate?

**The analytical approach:** Build and evaluate regression models to identify which variables predict monthly sales, quantify the relationship between each variable and revenue, and use predicted values to flag stores that are significantly outperforming or underperforming expectations.

---

## 2. Dataset Description

**File:** `data/business_regression_data.xlsx`
**Period:** January 2025 – April 2025 (4 months)
**Records:** 320 rows (80 stores × 4 months)
**Used in model:** 314 rows (6 excluded due to missing `competitor_distance_km`)

### Fields

| Field | Type | Range / Values | Description |
|---|---|---|---|
| `store_id` | String | STR-1001 to STR-1080 | Unique store identifier |
| `month` | Date | Jan–Apr 2025 | Observation month |
| `region` | Categorical | East, North, South, West | Geographic region |
| `store_type` | Categorical | Airport, High Street, Mall, Residential | Retail format |
| `marketing_spend` | Numeric | £9,535 – £172,416 | Monthly marketing budget |
| `footfall` | Integer | 971 – 12,870 | Monthly visitor count |
| `avg_discount_pct` | Numeric | 0% – 29% | Average discount applied |
| `staff_count` | Integer | 5 – 31 | Number of staff |
| `inventory_availability_pct` | Numeric | 71% – 99% | % of SKUs in stock |
| `competitor_distance_km` | Numeric | 0.20 – 22.02 km | Distance to nearest competitor |
| `holiday_flag` | Binary | 0 or 1 | 1 = month contains a public holiday |
| `customer_rating` | Numeric | 2.8 – 5.0 | Average customer satisfaction score |
| `monthly_sales` | Numeric | £400,832 – £946,834 | **Target variable** — total monthly revenue |
| `monthly_profit` | Numeric | £53,544 – £200,575 | Monthly profit (excluded — data leakage risk) |

### Data Quality

| Issue | Detail | Resolution |
|---|---|---|
| Missing `competitor_distance_km` | 6 records (1.9%) | Imputed with median (4.6 km) in cleaned dataset |
| Missing `customer_rating` | 8 records (2.5%) | Column excluded from model (near-zero correlation with sales) |
| Excluded: `store_id` | Identifier — no regression value | Dropped |
| Excluded: `month` | Only 4 levels; signal captured by `holiday_flag` | Dropped |
| Excluded: `monthly_profit` | Jointly determined with sales — data leakage | Dropped |
| Excluded: `staff_count` | r = 0.918 with footfall — severe multicollinearity | Dropped in favour of footfall |
| Excluded: `avg_discount_pct` | Endogenous — discounts applied reactively to struggling stores | Dropped |
| Excluded: `customer_rating` | r = −0.03 with sales — no predictive value | Dropped |

---

## 3. Dependent and Independent Variables

### Dependent Variable (Y)

**`monthly_sales`** — Total monthly store revenue in pounds sterling.

- Mean: £680,629
- Range: £400,832 to £946,834
- Distribution: near-symmetric (skewness = −0.18), no outliers — no transformation required

### Independent Variables in Final Model

| Variable | Type | Transformation | Correlation with Y | Rationale |
|---|---|---|---|---|
| `footfall` | Continuous | None | **r = 0.858** | Strongest single predictor |
| `marketing_spend` | Continuous | **log()** | r = 0.409 (raw) | Right-skewed (skew = 1.09), diminishing returns |
| `inventory_availability_pct` | Continuous | None | r = 0.115 | Operationally actionable — stockout cost |
| `competitor_distance_km` | Continuous | **log()** | r = −0.110 (raw) | Right-skewed (skew = 1.56), 6 missing imputed |
| `holiday_flag` | Binary | None | r = 0.195 | Seasonal control — already binary |
| `store_type` | Categorical | **One-hot encoded** | Varies by type | 4 formats → 3 dummy variables |

### Variables Tested but Excluded

| Variable | Reason for exclusion |
|---|---|
| `customer_rating` | r = −0.028, R² = 0.08%, p = 0.622 — no relationship with sales |
| `avg_discount_pct` | Endogenous — reflects underperformance, does not cause it |
| `staff_count` | r = 0.918 with footfall — would cause severe multicollinearity (VIF >> 5) |
| `monthly_profit` | Outcome variable — including it causes data leakage |

---

## 4. Regression Approach

### Models Tested

Three models were built and compared in sequence.

#### Model 1 — Simple Linear Regression (footfall)

```
Monthly Sales = £446,411 + £35.68 × footfall
```

- **R² = 73.6%** — strong single-variable result
- p < 0.001 — highly significant
- RMSE = £53,374
- Limitation: ignores marketing, inventory, store type, location, and seasonality

#### Model 2 — Simple Linear Regression (customer rating)

```
Monthly Sales = £703,801 − £5,446 × customer_rating
```

- **R² = 0.08%** — no explanatory power
- p = 0.622 — not statistically significant
- RMSE = £102,355 (worse than predicting the mean)
- Verdict: **eliminated** — customer rating is an outcome, not a driver of sales

#### Model 3 — Multiple Linear Regression (final model)

```
Monthly Sales = −£519,881
              + £33.04  × footfall
              + £66,202 × log(marketing_spend)
              + £3,056  × inventory_availability_pct
              − £8,295  × log(competitor_distance_km)
              + £12,113 × holiday_flag
              + £25,988 × store_type_Airport
              + £14,577 × store_type_Mall
              − £18,253 × store_type_Residential
```

- **R² = 83.5%** — most complete model
- Adjusted R² = 83.1% — improvement over Model 1 is genuine
- F-statistic = 193.25, p < 0.001
- RMSE = £42,966
- 7 of 8 predictors significant at α = 0.05
- All VIFs < 1.4 — no multicollinearity concern

### Statistical Tests Used

| Variable type | Test applied | Reason |
|---|---|---|
| Skewed continuous variables | Log transformation + OLS | Normalises distribution, linearises relationship |
| Categorical variables | One-hot encoding | Regression requires numeric input |
| Model significance | F-test | Tests whether all coefficients are jointly zero |
| Coefficient significance | t-test | Tests whether individual coefficient = 0 |
| Multicollinearity | VIF (Variance Inflation Factor) | All VIFs < 1.4 — no concern |

---

## 5. Dummy Variable Approach

### Why Dummy Variables Are Needed

`store_type` contains four categories (Airport, High Street, Mall, Residential). Regression models cannot work with text labels — each category must be converted into a binary (0/1) column.

### The Dummy Variable Trap

Creating four columns for four categories creates **perfect multicollinearity** — one column is always exactly predictable from the other three. This prevents the regression from finding a unique solution.

**Fix:** Drop one category (the reference/baseline). The remaining three dummy coefficients measure the sales *difference* from the baseline — not absolute sales.

### Encoding Applied

**`store_type`** — High Street as reference (largest group, n = 116, 36.3%):

| store_type | `store_type_Airport` | `store_type_Mall` | `store_type_Residential` |
|---|---|---|---|
| **High Street** ← reference | 0 | 0 | 0 |
| Airport | 1 | 0 | 0 |
| Mall | 0 | 1 | 0 |
| Residential | 0 | 0 | 1 |

**`region`** — East as reference (largest group, n = 104):
Three dummies created: `region_North`, `region_South`, `region_West`

### Reference Categories

| Variable | Reference | Reason |
|---|---|---|
| `store_type` | **High Street** | Largest group (n=116, 36.3%). Coefficients answer: *"How much more/less vs a High Street store?"* — a commercially meaningful comparison |
| `region` | **East** | Largest regional group (n=104). Provides the most statistically stable baseline estimate |

---

## 6. Model Comparison Summary

| Parameter | Model 1 — footfall | Model 2 — customer rating | Model 3 — multiple regression |
|---|---|---|---|
| Type | Simple Linear | Simple Linear | Multiple Linear |
| n | 320 | 312 | 314 |
| Predictors | 1 | 1 | 8 |
| R² | 73.6% | 0.08% | **83.5%** |
| Adjusted R² | 73.5% | −0.24% | **83.1%** |
| F-statistic | 887.85 ★★★ | 0.24 ns | **193.25 ★★★** |
| RMSE | £53,374 | £102,355 | **£42,966** |
| Significant variables | 1 of 1 | 0 of 1 | **7 of 8** |
| All VIFs < 1.4 | N/A | N/A | Yes ✓ |
| Business usefulness | High | None | **Very High** |
| Verdict | Runner-up | Eliminated | **Selected** |

> ★★★ p < 0.001 &nbsp; ★★ p < 0.01 &nbsp; ★ p < 0.05 &nbsp; ns = not significant

---

## 7. Final Model Selected

### Model 3 — Multiple Linear Regression

**Equation:**
```
Monthly Sales = −£519,881 + £33.04·footfall + £66,202·log(marketing_spend)
              + £3,056·inventory_pct − £8,295·log(competitor_dist)
              + £12,113·holiday_flag + £25,988·[Airport]
              + £14,577·[Mall] − £18,253·[Residential]
```

### Coefficient Interpretation (Business Language)

| Variable | Coefficient | What it means in practice |
|---|---|---|
| Intercept | −£519,881 | Mathematical anchor — no standalone business meaning |
| `footfall` ★★★ | +£33.04/visitor | Every 1,000 extra visitors → +£33,040/month |
| `log(marketing_spend)` ★★★ | +£66,202/log unit | Doubling a low-budget store's marketing → +£66K/month |
| `inventory_availability_pct` ★★★ | +£3,056/1pp | Going from 85%→95% stock availability → +£30,560/month |
| `log(competitor_distance_km)` ★★★ | −£8,295/log unit | Stores near competitors are in better commercial locations |
| `holiday_flag` (ns p=0.064) | +£12,113 | Holiday months earn ~£12K more — retain as control |
| `store_type_Airport` ★★ | +£25,988 | Airport stores earn £26K more/month vs High Street |
| `store_type_Mall` ★ | +£14,577 | Mall stores earn £15K more/month vs High Street |
| `store_type_Residential` ★★ | −£18,253 | Residential stores earn £18K less/month vs High Street |

### Why This Model Was Selected

1. **R² = 83.5%** — explains 9.9 percentage points more variance than footfall alone
2. **RMSE = £42,966** — average error £10,408 lower than Model 1; £59,389 lower than Model 2
3. **7 of 8 predictors significant** — robust, well-specified model
4. **All VIFs < 1.4** — no multicollinearity; coefficients are stable and interpretable
5. **Multi-dimensional** — captures five business levers simultaneously, each controlled for the others
6. **Directly actionable** — every coefficient maps to a decision leadership can take

### Residual Analysis Findings

- Mean residual = £0.21 ≈ 0 (OLS correctly centred ✓)
- **Positive residuals (outperformers):** 4 of top 5 are Residential stores — model under-predicts top-performing Residential locations, suggesting format heterogeneity
- **Negative residuals (underperformers):** Mall/East stores dominate — STR-1068 appears twice (Jan and Apr 2025), indicating a persistent structural issue
- **Key geographic bias:** East mean residual = −£15,148 (over-predicted); West = +£12,371 (under-predicted) — points to unmodelled regional dynamics

---

## 8. Business Recommendations

### Priority Actions

| Priority | Action | Evidence from model |
|---|---|---|
| 1 | Drive footfall at the 20 lowest-traffic stores | +1,000 visitors = +£33,040/month (p < 0.001) |
| 2 | Set 90% minimum inventory availability standard | Every 1pp shortfall costs ~£3,056/month (p < 0.001) |
| 3 | Redirect marketing budget to underinvested stores | Log relationship — diminishing returns at high spend |
| 4 | Site-visit top 10 positive residual outperformers | Earning £50–100K above prediction — find and replicate |
| 5 | Operational review of persistently negative residual stores | Mall/East stores failing to convert inputs into revenue |
| 6 | Collect 12 months of data and refresh model | Current 4-month window limits seasonality capture |

### What Not to Over-Interpret

- **Customer rating** — outcome of sales experience, not a driver. Do not use for revenue forecasting.
- **Discount %** — endogenous. Struggling stores discount more; discounting does not cause underperformance.
- **Competitor distance** — location quality proxy, not a competitive intensity measure.
- **Holiday flag** — borderline significance (p=0.064); directional guidance only with current 4-month data.

### Association vs Causation

The model shows what moves *together* with sales — it does not prove that changing footfall, marketing, or inventory will automatically produce the predicted uplift. The data is observational (not a controlled experiment), reverse causality is possible in several relationships, and omitted variables inflate apparent effects.

**Recommended approach:** Use coefficients to prioritise and set directional targets. Run a controlled pilot at 5 stores before committing estate-wide programmes. Measure actual outcomes against model predictions.

---

## 9. Assumptions and Limitations

### Statistical Assumptions

| Assumption | Status |
|---|---|
| Linear relationship (with log transforms) | Satisfied |
| Residuals approximately normally distributed | Satisfied — skewness = −0.01 |
| No severe multicollinearity | Satisfied — all VIFs < 1.4 |
| Mean residual ≈ 0 | Satisfied — mean residual = £0.21 |
| Homoscedasticity | Not formally tested — recommend Breusch-Pagan test with expanded dataset |

### Business Limitations

| Limitation | Impact | Mitigation |
|---|---|---|
| Only 4 months of data | Cannot capture seasonality, Q4 peaks, year-on-year trends | Collect 12+ months before next model refresh |
| East-West regional bias unexplained (±£12–15K mean) | Model systematically mis-predicts by region | Add regional economic / demographic index variable |
| 16.5% of variance unexplained | Manager quality, local events, product range not captured | Model complements judgment — does not replace it |
| Cross-sectional data only | Cannot isolate individual store effects | Panel regression with store fixed effects as data accumulates |
| Residential dummy hides format heterogeneity | Top Residential stores outperform prediction by £87–£101K | Add catchment density or urbanisation variable |
| No LTV or retention data | Revenue measured over 30 days only | Extend with 12-month trailing revenue per store |
| Model will become stale | Coefficients drift as market conditions change | Refresh annually at minimum |

---

## 10. Screenshots

All screenshots are stored in the `screenshots/` folder.

| File | Content |
|---|---|
| `simple_regression_output.png` | Model 1 (footfall) and Model 2 (customer rating) results — equations, R², p-values, business interpretation |
| `multiple_regression_output.png` | Full coefficient table — all 8 predictors with significance stars, confidence intervals, VIF values |
| `residuals_preview.png` | Top 5 positive and top 5 negative residuals with store-level business interpretation |
| `model_comparison_preview.png` | R² bar chart and RMSE comparison across all three models |

> Add screenshots by taking images of the Excel workbook outputs and saving them to the `screenshots/` folder with the filenames listed above.

---

## 11. Repository Structure

```
part3_regression_insights/
│
├── data/
│   └── business_regression_data.xlsx
│
├── analysis/
│   ├── regression_workbook.xlsx
│   ├── model_comparison.md
│   └── residual_analysis.md
│
├── outputs/
│   ├── business_regression_data_CLEANED.xlsx
│   ├── Simple_Regression_Results.xlsx
│   ├── Multiple_Regression_Results.xlsx
│   ├── Model_Comparison_Table.xlsx
│   ├── Residual_Analysis.xlsx
│   ├── dummy_variable_approach.md
│   ├── model_equations.md
│   ├── residual_analysis_summary.md
│   └── final_recommendation.md
│
├── screenshots/
│   ├── simple_regression_output.png
│   ├── multiple_regression_output.png
│   ├── residuals_preview.png
│   └── model_comparison_preview.png
│
└── README.md
```

---

## Quick Reference

```
Dataset           : business_regression_data.xlsx
Stores            : 80 unique stores
Observation period: January – April 2025 (4 months)
Records in model  : 314 (post-cleaning)
Dependent variable: monthly_sales  (£401K – £947K, mean £681K)
Final model       : Multiple Linear Regression (8 predictors)
R²                : 83.5%
Adjusted R²       : 83.1%
RMSE              : £42,966  (~6.3% of mean sales)
Significant vars  : 7 of 8  (holiday_flag borderline p=0.064)
All VIFs          : < 1.4   (no multicollinearity)
Top driver        : footfall  (+£33.04 per additional visitor)
Reference category: High Street (store_type),  East (region)
```

---

*Prepared by: Business Analysis Team — June 2026*
*Project: `part3_regression_insights` | Dependent variable: `monthly_sales`*

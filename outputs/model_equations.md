# Dummy Variable Encoding — `store_type` and `region`

## What is a dummy variable?

Regression models require all inputs to be numeric. `store_type` and `region` are **categorical variables** — they describe group membership, not quantity. You cannot multiply "High Street" by a coefficient. To use them in regression, each category is converted into a binary column (0 or 1) called a **dummy variable**.

A dummy variable equals `1` if the observation belongs to that category, and `0` if it does not.

---

## The dummy variable trap — why k−1 columns, not k

If `store_type` has 4 categories (Airport, High Street, Mall, Residential), the naive approach is to create 4 binary columns. This is a mistake.

With 4 columns, one column is always perfectly predictable from the other three:

```
store_type_Residential = 1 − store_type_Airport − store_type_High_Street − store_type_Mall
```

This **perfect multicollinearity** makes the regression matrix singular — the system has no unique solution and the model cannot estimate coefficients. This is called the **dummy variable trap**.

The solution is to create **k−1 columns**, dropping one category entirely. The dropped category becomes the **reference (baseline) category**. Its effect is absorbed into the model intercept. The remaining dummy coefficients measure the difference from the baseline, not the absolute level.

---

## Encoding applied in this dataset

### `store_type` — 4 categories → 3 dummy columns

| Original value | `store_type_Airport` | `store_type_Mall` | `store_type_Residential` |
|---|---|---|---|
| **High Street** | 0 | 0 | 0 |
| Airport | 1 | 0 | 0 |
| Mall | 0 | 1 | 0 |
| Residential | 0 | 0 | 1 |

**Reference category: High Street**

When all three dummy columns equal 0, the observation is a High Street store. The model intercept represents the predicted sales for a High Street store (at the mean of all other predictors).

### `region` — 4 categories → 3 dummy columns

| Original value | `region_North` | `region_South` | `region_West` |
|---|---|---|---|
| **East** | 0 | 0 | 0 |
| North | 1 | 0 | 0 |
| South | 0 | 1 | 0 |
| West | 0 | 0 | 1 |

**Reference category: East**

---

## Why High Street is the reference category for `store_type`

The choice of reference category does not affect model fit (R², RMSE, F-statistic) — these are mathematically identical regardless of which category is dropped. What changes is the **interpretation** of every coefficient for that variable. High Street was selected for four reasons:

### 1. It is the most common category

| Store type | Count | Share |
|---|---|---|
| **High Street** | **116** | **36.3%** |
| Residential | 100 | 31.3% |
| Mall | 76 | 23.8% |
| Airport | 28 | 8.8% |
| **Total** | **320** | **100%** |

Coefficients for Airport, Mall, and Residential are estimated relative to High Street. The larger the baseline group, the more precisely estimated the baseline mean, and the smaller the standard errors on the dummy coefficients. Airport has only 28 observations — if it were the baseline, all three dummy coefficients would inherit its relatively high estimation uncertainty.

### 2. It is the most representative "typical" store

High Street stores represent the standard retail environment the business operates in most broadly. Using it as the baseline means dummy coefficients answer the practically useful question: *"How much more or less does this store type sell compared to a typical High Street store?"*

If Airport were the baseline, the coefficients would answer: *"How much less does a High Street, Mall, or Residential store sell compared to an airport location?"* — a less useful framing since airport stores are atypical and operate under very different conditions (captive audience, higher prices, restricted competition).

### 3. It has the lowest average sales — making uplifts interpretable as positive

| Store type | Avg monthly sales |
|---|---|
| **High Street** | **£667,010** |
| Residential | £673,773 |
| Mall | £697,425 |
| Airport | £715,944 |

With High Street as the baseline, all three dummy coefficients are expected to be **positive** — each other store type outperforms High Street on average. Positive coefficients are easier to communicate to stakeholders than a mix of positive and negative values that depend on an arbitrary baseline choice.

### 4. It avoids zero-frequency problems in interaction terms

If the dataset were extended to include interaction terms (e.g., `store_type × holiday_flag`), a small baseline group like Airport (n=28) creates near-empty cells in the interaction matrix. High Street (n=116) is robust to this.

---

## How to interpret the dummy coefficients in regression output

Suppose the fitted model produces these coefficients (illustrative):

```
Intercept             =  500,000
store_type_Airport    =  +42,000
store_type_Mall       =  +24,000
store_type_Residential =  +8,000
```

**Reading these:**

- A **High Street** store is predicted to generate the intercept value (£500,000 at mean predictors)
- An **Airport** store is predicted to generate £500,000 + £42,000 = **£542,000** — £42,000 more than a comparable High Street store
- A **Mall** store generates £500,000 + £24,000 = **£524,000**
- A **Residential** store generates £500,000 + £8,000 = **£508,000**

Each coefficient is the **ceteris paribus** (all else equal) sales differential vs. High Street, controlling for footfall, marketing spend, holiday periods, inventory, competitor distance, and region.

---

## Why East is the reference category for `region`

East was selected as the baseline for `region` because it has the largest number of observations (n=104), following the same logic of maximising baseline estimation precision. The regional sales differences are modest (range: £669K–£693K), so the choice of baseline has limited practical impact on interpretation, but East provides the most stable foundation.

| Region | Count | Avg monthly sales |
|---|---|---|
| **East** | **104** | **£680,382** |
| West | 92 | £680,681 |
| North | 64 | £669,056 |
| South | 60 | £693,320 |

---

## Summary

| Variable | Categories | Dummies created | Reference category | Reason for reference choice |
|---|---|---|---|---|
| `store_type` | 4 | 3 | **High Street** | Most common (n=116); most representative; lowest avg sales → uplifts read as positive |
| `region` | 4 | 3 | **East** | Largest group (n=104); stable baseline estimation |

The total number of columns added to the model from encoding: **3 + 3 = 6 binary columns**, replacing 2 categorical columns. The model-ready dataset therefore uses these dummies directly and excludes the original `region` and `store_type` text columns.

---

*Part of: `part3_regression_insights` | Analysis: `regression_workbook.xlsx` | June 2026*


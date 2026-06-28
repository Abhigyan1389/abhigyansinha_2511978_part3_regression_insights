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
**Model Equations**
# Regression Model Equations — Business Guide

**Project:** Retail Store Sales Analysis  
**Dataset:** `business_regression_data.xlsx` — 80 stores × 4 months (Jan–Apr 2025)  
**Goal:** Understand what drives monthly store sales and build a model to predict and benchmark performance

---

## 1. Simple Regression Equations

Two simple models were tested first — one variable at a time — to understand individual relationships before building a fuller model.

### Model 1 — Footfall predicts monthly sales

```
Monthly Sales = £446,411 + £35.68 × Footfall
```

**Plain English:** A store that attracts zero walk-in visitors would still generate an estimated £446,411 per month (from online, phone, or regular orders). For every additional visitor who walks through the door, monthly sales increase by approximately £35.68.

**What this tells you:** If your store currently receives 6,000 visitors per month and you run a campaign that drives footfall up by 1,000 extra visitors, you can expect roughly **£35,680 more in monthly revenue**. Footfall is by far the single biggest lever in the dataset.

| | Value |
|---|---|
| R² (variance explained) | **73.6%** |
| Average prediction error (RMSE) | £53,374 |
| Is footfall statistically significant? | Yes — p < 0.001 |
| Confidence interval for slope | £33.32 to £38.03 per visitor |

---

### Model 2 — Customer rating predicts monthly sales

```
Monthly Sales = £703,801 − £5,446 × Customer Rating
```

**Plain English:** The equation suggests sales fall by £5,446 for every one-point increase in customer rating — which makes no business sense at face value. A store with a perfect 5.0 rating should not sell less than a store with a 3.0 rating.

**What this tells you:** Customer rating does not predict sales. The relationship is essentially flat (r = −0.028), and the result is **not statistically significant** (p = 0.622). The confidence interval for the slope runs from −£27,176 to +£16,284 — it crosses zero, meaning we cannot even reliably determine whether the relationship is positive or negative.

The likely reason: customer rating reflects how good the shopping experience was *after* the sale. It is a consequence of high sales and good service, not a cause of it. Using this variable to predict revenue would be misleading.

| | Value |
|---|---|
| R² (variance explained) | **0.08%** — negligible |
| Average prediction error (RMSE) | £102,355 |
| Is rating statistically significant? | No — p = 0.622 |
| Verdict | Excluded from all models |

---

## 2. Multiple Regression Equation

After testing individual variables, all useful predictors were combined into one model.

```
Monthly Sales = −£519,881
              + £33.04  × Footfall
              + £66,202 × Log(Marketing Spend)
              + £3,056  × Inventory Availability (%)
              − £8,295  × Log(Competitor Distance km)
              + £12,113 × Holiday Month (0 or 1)
              + £25,988 × Airport Store (0 or 1)
              + £14,577 × Mall Store (0 or 1)
              − £18,253 × Residential Store (0 or 1)
```

This model explains **83.5%** of the variation in monthly sales across all 314 store-month observations. The average prediction error is **£42,966** — about 6.3% of the average store's monthly revenue.

---

## 3. Coefficient Explanations — In Business Language

### Intercept: −£519,881

This is the mathematical starting point of the model. It does not represent a real-world scenario (a store with zero footfall, no marketing budget, no inventory, and no nearby competitors does not exist). Think of it as a calibration anchor rather than a meaningful business number. In practice, when you plug realistic values for each variable into the equation, the predictions land close to actual store sales.

---

### Footfall: +£33.04 per additional visitor

**What it means:** Every additional person who visits a store in a month is associated with roughly **£33 more in monthly sales**, holding everything else constant.

**Business implication:** Footfall is the single most powerful driver in the model. Initiatives that increase foot traffic — window displays, local events, social media campaigns, outdoor advertising, or partnerships with nearby businesses — deliver a directly measurable and statistically confirmed sales return. A drive to add 2,000 extra visitors per month to a typical store is expected to generate approximately **£66,000 in additional monthly revenue**.

**Confidence level:** Very high. p < 0.001. The estimate is precise: we are 95% confident the true effect is between £31.03 and £35.05 per visitor.

---

### Log(Marketing Spend): +£66,202 per unit increase

**What it means:** Marketing spend has a positive but *diminishing returns* relationship with sales. The log transformation means that doubling your marketing budget from a low base delivers a bigger boost than doubling it from a high base.

**Business implication:** Stores currently spending very little on marketing (below £30,000 per month) gain the most from additional investment. For these stores, increasing the budget meaningfully — say from £20,000 to £54,000 (roughly a 2.72× increase, or one log unit) — is associated with approximately **£66,200 more in monthly sales**. For stores already spending £100,000+, additional spend delivers proportionally less. This supports a **tiered marketing budget strategy**: increase spend at underperforming, low-investment stores first rather than applying flat increases across all locations.

**Confidence level:** Very high. p < 0.001.

---

### Inventory Availability (%): +£3,056 per 1 percentage point improvement

**What it means:** For every 1 percentage point increase in the share of products that are actually in stock, monthly sales increase by approximately **£3,056**.

**Business implication:** Stockouts cost money in a directly measurable way. A store operating at 85% inventory availability — below the dataset mean of 88.3% — is predicted to generate roughly **£9,168 less per month** than a comparable store at 88%. Improving from 85% to 95% availability (a realistic supply chain improvement) is associated with approximately **£30,560 more per month**. Maintaining availability above 90% should be a minimum operational standard. This coefficient also underscores that the investment in better stock management and supplier relationships pays for itself in revenue.

**Confidence level:** Very high. p < 0.001. CI: £2,149 to £3,963.

---

### Log(Competitor Distance): −£8,295 per unit increase

**What it means:** Stores that are further away from their nearest competitor generate **lower** monthly sales — even after accounting for footfall, marketing, and store type. This is counterintuitive at first glance.

**Business implication:** Being far from a competitor is not an advantage in this dataset — it is a disadvantage. The reason is a **location quality effect**: stores located close to competitors tend to be in busy commercial areas (city centres, shopping districts, high streets) that naturally attract more and better-quality footfall. Isolated stores sit in quieter locations where overall retail spending is lower. The variable is acting as a proxy for commercial density. The practical message for site selection teams: **prioritise high-footfall commercial clusters over isolated standalone locations**, even if that means facing nearby competition. Being surrounded by other retailers usually signals a better trading environment.

**Confidence level:** High. p < 0.001. CI: −£12,429 to −£4,161.

---

### Holiday Month Flag: +£12,113 when a public holiday falls in the month

**What it means:** Months containing a public holiday are associated with approximately **£12,113 more in monthly sales** compared to non-holiday months.

**Business implication:** Holiday periods drive additional footfall and spending. Stores should plan promotional activity, stock levels, and staffing around holiday months to capture this uplift. However, this coefficient is **borderline statistically significant** (p = 0.064 — just above the 5% threshold). With only 4 months of data, the estimate is underpowered. As more data is collected, this effect will likely become firmly significant. For now, treat the £12,113 as a directional estimate that supports holiday planning, not a precise forecast.

---

### Store Type — Airport: +£25,988 vs a High Street store

**What it means:** An Airport store generates approximately **£25,988 more per month** than a comparable High Street store, after accounting for footfall, marketing, inventory, and location.

**Business implication:** Airport locations command a structural revenue premium even after controlling for how busy they are. This reflects higher average transaction values (travel necessities, impulse purchases, fewer price comparisons), captive customer pools with limited alternatives, and a customer mindset more inclined toward spending. Airport stores merit different cost structures, product ranges, and pricing strategies than High Street formats. They should be benchmarked against each other, not against High Street peers.

**Confidence level:** High. p = 0.005.

---

### Store Type — Mall: +£14,577 vs a High Street store

**What it means:** A Mall store generates approximately **£14,577 more per month** than a comparable High Street store.

**Business implication:** Mall locations benefit from concentrated shopper dwell time, a surrounding retail ecosystem that drives browsing behaviour, and customers who arrive with a clear intent to shop. The premium is smaller than Airport but more widely applicable (76 Mall stores vs 28 Airport stores in the dataset). Lease negotiations and operational investment decisions for Mall locations should reflect this structural advantage.

**Confidence level:** Moderate. p = 0.025.

---

### Store Type — Residential: −£18,253 vs a High Street store

**What it means:** A Residential store generates approximately **£18,253 less per month** than a comparable High Street store.

**Business implication:** Residential locations serve smaller, more local catchment areas. Customers visiting a Residential store typically have a specific purpose rather than browsing intent, which tends to produce lower average basket sizes and fewer impulse purchases. The structural disadvantage persists even after accounting for actual footfall — meaning the nature of Residential shopping behaviour, not just visitor volume, suppresses revenue. This has direct implications for cost management: Residential store cost bases (rent, staffing, inventory investment) should be calibrated to lower expected revenues than High Street formats.

**Confidence level:** High. p = 0.002.

---

## 4. Dummy Variables Explained

### What is a dummy variable?

A dummy variable is a binary switch — it equals **1** if an observation belongs to a particular category, and **0** if it does not.

Regression models can only work with numbers. `store_type` is a label (Airport, High Street, Mall, Residential) — not a number. To include it in the model, each category is converted into its own 0/1 column.

### Why not create a column for every category?

If you create four dummy columns for four store types, one column is always perfectly predictable from the other three. For example:

```
store_type_High Street = 1 − store_type_Airport − store_type_Mall − store_type_Residential
```

This creates a mathematical problem called **perfect multicollinearity** — the regression cannot produce a unique solution. The fix is to drop one category. The dropped category becomes the **reference** (or baseline). Instead of measuring absolute sales for each store type, the model measures the *difference* from the baseline.

### The dummy variables used in this model

For `store_type`, three dummy columns were created:

| store_type | store_type_Airport | store_type_Mall | store_type_Residential |
|---|---|---|---|
| **High Street** (reference) | 0 | 0 | 0 |
| Airport | 1 | 0 | 0 |
| Mall | 0 | 1 | 0 |
| Residential | 0 | 0 | 1 |

When all three dummies equal 0, the store is a High Street store. The model intercept captures its baseline.

For `region`, the same approach was applied (East as the reference, three dummy columns for North, South, and West).

---

## 5. Reference Categories

| Variable | Reference Category | Why This Category Was Chosen |
|---|---|---|
| `store_type` | **High Street** | Largest group in the dataset (n = 116, 36.3% of all stores). Using the most common category as the baseline gives the most statistically stable reference point and means coefficients are easiest to interpret: each dummy coefficient tells you how much more or less an Airport, Mall, or Residential store earns compared to a typical High Street store — a commercially meaningful comparison. |
| `region` | **East** | Largest regional group (n = 104 store-months, 33.1% of observations). Same logic: the biggest group provides the most precise baseline estimate. Regional dummies for North, South, and West measure sales differences relative to East-region stores. |

---

## 6. Final Model Selected

**Selected model: Model 3 — Multiple Linear Regression**

```
Monthly Sales = −£519,881 + £33.04·Footfall + £66,202·Log(Marketing Spend)
              + £3,056·Inventory (%) − £8,295·Log(Competitor Distance)
              + £12,113·Holiday Month + £25,988·[Airport]
              + £14,577·[Mall] − £18,253·[Residential]
```

| Performance metric | Model 3 (selected) | Model 1 (footfall only) | Model 2 (customer rating) |
|---|---|---|---|
| R² | **83.5%** | 73.6% | 0.08% |
| Adjusted R² | **83.1%** | 73.5% | −0.24% |
| RMSE | **£42,966** | £53,374 | £102,355 |
| Significant predictors | **7 of 8** | 1 of 1 | 0 of 1 |
| Verdict | **Selected** | Runner-up | Eliminated |

---

## 7. Reasons for Selecting the Final Model

### A. It explains the most about what drives sales

Model 3 explains **83.5% of the variation in monthly sales** across 314 store-month observations. That means if you look at why one store generates £600,000 and another generates £900,000, this model accounts for roughly 83 pence of every £1 of that difference. The simple footfall model explains 73.6% — still strong, but leaving significantly more unexplained. Customer rating explains almost nothing (0.08%).

### B. It is the most accurate predictor

The average prediction error (RMSE) for Model 3 is **£42,966** — approximately 6.3% of average monthly sales. For context:

- Model 1 (footfall only): average error of £53,374 — 24% less accurate
- Model 2 (customer rating): average error of £102,355 — more than twice as inaccurate

When used to benchmark a store's performance, Model 3 gives a tighter and more reliable estimate of what a store *should* be generating given its inputs.

### C. Seven of eight variables are statistically confirmed

Seven of the eight predictors are significant at the 5% level — meaning the chance that their coefficients are zero (no real effect) is less than 5%. The one borderline predictor, `holiday_flag` (p = 0.064), is retained as a control variable because it is theoretically important and nearly significant despite the limited four-month dataset. All eight variables have no multicollinearity concerns (all VIFs below 1.4), meaning the coefficients are stable and reliable.

### D. It captures multiple business dimensions simultaneously

Model 1 (footfall only) measures one lever. Model 3 measures **eight dimensions at once** — and because it controls for all of them together, each coefficient gives a cleaner, fairer answer. For example:

- The footfall coefficient in Model 3 (£33.04) tells you the sales return from one extra visitor *after accounting for marketing spend, inventory, location, and store type*
- Without those controls, the footfall-only model's coefficient (£35.68) partially absorbs the effects of all the variables it is not measuring

Model 3 coefficients are closer to the true causal effects. Model 1 coefficients are contaminated by omitted variables.

### E. Every coefficient is directly actionable

Each variable in Model 3 corresponds to a decision a business can make:

| Variable | Decision it informs |
|---|---|
| Footfall | Footfall-driving investment: events, campaigns, visual merchandising |
| Marketing spend | Budget allocation — how much to invest, and where returns are highest |
| Inventory availability | Stock management targets and supply chain investment |
| Competitor distance | Site selection — commercial density vs isolation |
| Holiday flag | Promotional planning and seasonal staffing |
| Store type dummies | Format strategy, cost structure, and performance benchmarks by format |

Model 1 only informs one of these decisions. Model 2 informs none.

### F. Customer rating was correctly excluded

Customer rating was not included in the final model because it has no statistically significant relationship with monthly sales (p = 0.622, R² = 0.08%). More importantly, customer rating is an **outcome**, not a driver — stores that provide a good experience generate high ratings after the fact. Including it in a sales prediction model would be circular reasoning and would add noise without improving predictions.

---

## Summary

The multiple regression model was selected because it is the most complete, most accurate, and most actionable model tested. It explains 83.5% of sales variance, predicts monthly revenue within ±£43K on average, uses eight variables that are all theoretically grounded and individually interpretable, and translates directly into operational decisions across footfall, marketing, inventory, location, and store format strategy.

The simple footfall model remains a useful communication tool for stakeholders who want a single number: every additional visitor is worth approximately £33–£36 in monthly sales. But for planning, benchmarking, and diagnosing store performance, the multiple regression model is the definitive analytical tool for this dataset.

---

*Part of: `part3_regression_insights` | Dependent variable: monthly_sales | n = 314 observations | June 2026*


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


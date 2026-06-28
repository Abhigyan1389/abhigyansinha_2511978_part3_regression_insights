# Residual Analysis — Multiple Linear Regression Model

## Overview

This document summarises the residual analysis performed on the final selected multiple linear regression model predicting monthly store sales across 314 store-month observations (80 stores × 4 months, January–April 2025, excluding 6 rows with missing `competitor_distance_km`).

**Model equation:**
```
Monthly Sales = −519,881 + 33.04·footfall + 66,202·log(marketing_spend)
              + 3,056·inventory_availability_pct − 8,295·log(competitor_distance_km)
              + 12,113·holiday_flag + 25,988·[Airport] + 14,577·[Mall] − 18,253·[Residential]
```

**Residual definition:** `Residual = Actual Sales − Predicted Sales`

---

## Residual Summary Statistics

| Statistic | Value |
|---|---|
| Number of observations | 314 |
| Mean residual | £0.21 (≈ 0 — OLS correctly centred) |
| Standard deviation | £42,414 |
| RMSE | £42,966 |
| Minimum residual | −£149,947 (STR-1017, Mar-2025) |
| Maximum residual | +£101,286 (STR-1030, Feb-2025) |
| Residuals within ±£25K | 146 of 314 (46.5%) |
| Skewness | −0.01 (approximately symmetric) |

The mean residual of £0.21 ≈ 0 confirms OLS is working correctly — residuals are centred at zero as required.

---

## Top 5 Largest Positive Residuals

> A positive residual means actual sales **exceeded** the model prediction. The store outperformed what its observable inputs would predict.

| Rank | Store | Month | Store Type | Region | Actual Sales | Predicted | Residual | % Over |
|---|---|---|---|---|---|---|---|---|
| 1 | STR-1030 | Feb 2025 | Residential | West | £820,519 | £719,233 | **+£101,286** | +12.3% |
| 2 | STR-1073 | Mar 2025 | Residential | East | £813,317 | £716,885 | **+£96,431** | +11.9% |
| 3 | STR-1050 | Apr 2025 | Residential | North | £735,787 | £646,120 | **+£89,667** | +12.2% |
| 4 | STR-1059 | Feb 2025 | Residential | West | £723,305 | £634,166 | **+£89,140** | +12.3% |
| 5 | STR-1041 | Feb 2025 | High Street | South | £778,299 | £691,217 | **+£87,082** | +11.2% |

### Business meaning

These stores sold significantly more than their footfall, marketing spend, inventory, location, and format would predict. The gap is caused by an **unmodelled outperformance driver** — something real and impactful that the eight predictors do not capture.

Likely causes include:
- A highly effective store manager or team driving conversion above the norm
- A uniquely loyal local customer base generating high repeat-purchase frequency
- A co-located anchor (transport hub, leisure venue, food outlet) driving incidental footfall
- A product range specifically calibrated to the local demographic
- Micro-location advantages (corner position, high-visibility signage) not in the dataset

These stores are **best-practice candidates**. They should be studied to identify the replicable factors behind their outperformance so those factors can be applied to similar stores.

---

## Top 5 Largest Negative Residuals

> A negative residual means actual sales **fell short** of the model prediction. The store underperformed what its observable inputs would predict.

| Rank | Store | Month | Store Type | Region | Actual Sales | Predicted | Residual | % Under |
|---|---|---|---|---|---|---|---|---|
| 1 | STR-1017 | Mar 2025 | High Street | West | £685,379 | £835,326 | **−£149,947** | −21.9% |
| 2 | STR-1068 | Jan 2025 | Mall | East | £660,634 | £768,949 | **−£108,315** | −16.4% |
| 3 | STR-1068 | Apr 2025 | Mall | East | £593,343 | £698,507 | **−£105,164** | −17.7% |
| 4 | STR-1052 | Apr 2025 | Mall | East | £675,918 | £771,118 | **−£95,201** | −14.1% |
| 5 | STR-1001 | Apr 2025 | Residential | East | £658,920 | £750,534 | **−£91,614** | −13.9% |

### Business meaning

These stores failed to convert their observable inputs (footfall, marketing, inventory) into the level of revenue the model expected. Key observations:

- **STR-1017 (High Street/West, −£150K):** Very high footfall (9,628), perfect inventory (99%), holiday month — model predicts £835K. Actual only £685K. The store sits 19.0 km from the nearest competitor — the model treats this positively (isolated location), but in practice the store appears to struggle with conversion despite strong traffic. The isolation may reflect a low-quality catchment rather than a competitive advantage.

- **STR-1068 (Mall/East):** Appears **twice** in the top 5 (January and April 2025) with a competitor at 0.65 km and 0.20 km respectively. The Mall dummy captures an average mall premium, but cannot account for the severe competitive pressure at this specific location. This store is a **persistent underperformer** and warrants urgent operational review.

- **STR-1052 (Mall/East, −£95K):** A different Mall/East store showing the same pattern. High footfall (8,303) and good inventory (93.6%) but actual sales far below prediction. Suggests a broader Mall/East dynamic, not just an isolated store issue.

- **STR-1001 (Residential/East, −£92K):** High footfall (9,431) but below-predicted sales. High traffic that does not convert — points to poor product-market fit, below-average basket size, or an unrecorded competitive threat nearby.

---

## What These Residuals Mean in Business Terms

### Positive residuals → investigate for best practice

Large positive residuals identify stores operating above their predicted ceiling. These gaps represent **unexplained competitive advantages** — real value drivers that the model cannot see. For operations and strategy teams, these are the stores to study first. The questions to ask are:

1. What is different about this store's management, layout, or product mix?
2. Does the local catchment have unusually high loyalty or fewer alternative shopping options?
3. Are there unrecorded location attributes (proximity to schools, offices, stations) that drive traffic quality?
4. Can the store's operational practices be documented and replicated elsewhere?

### Negative residuals → investigate for operational issues

Large negative residuals identify stores that are **wasting their inputs**. High footfall that does not convert to revenue is the most common pattern — traffic is arriving but customers are not buying. The questions to ask are:

1. What is the in-store conversion rate (visitors who purchase ÷ total visitors)?
2. Is the product range matched to the local demographic?
3. Is there an unrecorded competitor (online, market, pop-up) drawing spend away?
4. Are there physical barriers to purchase (poor layout, long queues, limited payment options)?

---

## Model Bias — Does the Model Over- or Under-Predict Certain Store Types?

### By store type — no systematic bias

| Store Type | Mean Residual | Median Residual | n |
|---|---|---|---|
| Airport | +£0.22 | +£1,726 | 28 |
| High Street | +£0.19 | −£2,197 | 111 |
| Mall | +£0.21 | +£1,465 | 75 |
| Residential | +£0.21 | −£374 | 100 |

All four store types have mean residuals of approximately £0 — OLS is **unbiased on average** across all store types. The slight negative median for High Street (−£2,197) indicates the model marginally over-predicts typical High Street stores, but this is small relative to RMSE of £42,966 and is not operationally significant. No store type is systematically mispriced by the model.

### By region — significant East-West bias (most important finding)

| Region | Mean Residual | Median Residual | n | Assessment |
|---|---|---|---|---|
| East | −£15,148 | −£11,985 | 102 | **Model over-predicts** East stores |
| North | −£1,764 | −£7,161 | 63 | Near-neutral, slight over-prediction |
| South | +£9,203 | +£10,920 | 59 | Model under-predicts South stores |
| West | +£12,371 | +£14,961 | 90 | **Model under-predicts** West stores |

The East-West regional bias is the **clearest diagnostic signal** in the entire residual analysis:

- The model **over-predicts East-region stores** by an average of −£15,148 per store-month
- The model **under-predicts West-region stores** by an average of +£12,371 per store-month

This bias exists despite the regional dummies in the model. The dummies capture average sales-level differences between regions — but they cannot capture regional differences in *how* each predictor operates. For example, the same level of footfall or marketing spend may translate into different sales outcomes in East vs West due to differences in local economic conditions, demographic spending power, or regional competitive intensity. These are not captured in the current eight-variable model.

### The Residential format anomaly

4 of the top 5 positive residuals are Residential-format stores. The model applies a flat **−£18,253 structural discount** to all Residential stores relative to High Street (the baseline). This discount correctly captures the average underperformance of Residential stores. However, the top-performing Residential stores overcome this penalty by £87K–£101K — pointing to a pronounced ceiling effect. Dense urban Residential locations with captive local shoppers behave like a distinct sub-format. The single dummy coefficient cannot distinguish them from low-footfall suburban Residential stores that genuinely underperform.

---

## Model Improvement Recommendations Based on Residuals

| Finding | Improvement |
|---|---|
| East over-predicted by mean −£15K, West under-predicted by mean +£12K | Add region-specific economic or demographic variable (e.g. regional consumer confidence index, postcode-level population density). Consider region × footfall interaction term. |
| Top Residential stores outperform prediction by £87K–£101K | Add catchment density or urbanisation variable to differentiate dense urban Residential from suburban Residential. Consider a Residential × catchment interaction. |
| STR-1068 (Mall/East) appears in negative residuals twice — competitor at 0.20 km | Add competitor count within 1 km radius as a supplementary variable alongside single nearest-competitor distance. |
| STR-1017: very high footfall (9,628), 99% inventory — yet £150K below prediction | Add a conversion rate variable or basket size proxy if POS data is available. Footfall is a necessary but not sufficient condition for sales. |
| Holiday flag borderline (p=0.064) | Collect 12+ months of data. The holiday coefficient is statistically underpowered with only 4 months of observations. |

---

## Summary

The residual analysis confirms that the multiple regression model is well-specified on average — OLS produces residuals centred at zero across all store types. The model explains 83.5% of sales variance and predicts within ±£25K for 46.5% of observations.

The two most actionable findings are:

1. **The East-West regional bias** — a systematic and material prediction error (±£12–15K per store-month) that points to unmeasured regional structural differences not captured by the current regional dummies.

2. **The Residential ceiling effect** — the model correctly prices average Residential stores but systematically under-predicts high-performing Residential stores in dense urban locations, suggesting the format is more heterogeneous than a single dummy coefficient can represent.

Both findings point toward the same model improvement: collect richer location-level and demographic data to distinguish within-format and within-region variation that is currently absorbed into the residuals.

---

*Part of: `part3_regression_insights` | Model: Multiple Linear Regression | n = 314 | June 2026*

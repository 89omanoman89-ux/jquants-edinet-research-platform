# P0 trend-quality / moving-average audit — 2026-09-19

## Question

Do moving-average position, trend direction, and trend stability add next-day predictive information beyond the frozen P0 score?

This is a secondary audit on the independent 120-stock sample already selected for the private tick package. No licensed raw data is committed to GitHub.

## Sample and split

The 120-stock sample was selected in 2024-08 without return-based selection.

For this audit:

- feature history begins 2024-05 to provide the longest 60-day lookback
- train: 2024-09 through 2025-06
- validation/model selection: 2025-07 through 2025-09
- untouched test for this audit: 2025-10 through 2026-08-25
- test days: 218
- joined stocks: 119

The baseline input is the already frozen full-universe P0 percentile score. Trend features are computed from information available by the signal-day close.

## Pre-specified feature blocks

### Moving-average geometry

- close / MA5 distance
- close / MA20 distance
- close / MA60 distance
- MA5 / MA20
- MA20 / MA60
- 5-day change in MA20
- 20-day change in MA60

### Trend direction / quality

- 20-day and 60-day log-price regression slope
- 20-day and 60-day regression R2
- signed 20-day and 60-day Efficiency Ratio
- 20-day and 60-day fraction of positive-return days
- 20-day persistence above MA20
- 20-day and 60-day return / volatility trend strength

The purpose is to separate:

1. direction,
2. distance from trend,
3. trend persistence,
4. trend smoothness.

## Model-level result

### Conservative Ridge integration

| Model | Test mean daily IC | Delta vs frozen P0 | Top-10% excess |
|---|---:|---:|---:|
| frozen P0 | **0.04736** | — | +1.26 bp/day |
| P0 + MA block | 0.04655 | -0.00081 | +1.42 bp/day |
| P0 + trend-quality block | 0.03236 | -0.01500 | +4.17 bp/day |
| P0 + full trend block | 0.03617 | -0.01119 | -3.32 bp/day |

Paired test for the MA block versus frozen P0:

- mean daily IC delta: -0.00081
- t-stat: -0.13
- top-decile delta: +0.16 bp/day
- top-decile t-stat: 0.04

Therefore a generic moving-average block does **not** add robust incremental forecasting power in this test.

### Nonlinear LightGBM integration

The nonlinear secondary models overfit strongly.

For example, P0 + MA:

- train IC: 0.179
- validation IC: 0.043
- test IC: 0.0188

The full trend nonlinear block also underperforms frozen P0 on test.

This is useful negative evidence: many overlapping trend transformations are easy to overfit in a small cross-sectional sample.

## Regime instability

The MA Ridge result changes sign across subperiods.

| Period | Frozen P0 IC | P0 + MA Ridge IC |
|---|---:|---:|
| 2025 Q4 | 0.0600 | **0.0761** |
| 2026 Jan-Aug | **0.0423** | 0.0348 |

The same block that appears useful in 2025 Q4 hurts in 2026.

Do not interpret moving-average geometry as a stable unconditional alpha.

## Individual-feature findings

Although the full block fails, some individual long-horizon trend variables have nonzero association with the next-day cross section.

Untouched-test standalone mean daily rank IC:

| Feature | Mean IC | t-stat |
|---|---:|---:|
| 60-day positive-day fraction | **+0.0323** | **3.13** |
| close / MA5 distance | **-0.0316** | **-2.67** |
| MA20 / MA60 | +0.0179 | 1.60 |
| 60-day trend slope | +0.0178 | 1.62 |
| close / MA20 distance | -0.0169 | -1.48 |

Interpretation:

- short-horizon overextension above the fast MA tends to mean-revert the next day;
- longer-horizon trend persistence tends to continue.

This is a horizon interaction, not a simple “trend is good” result.

## Incremental association after controlling frozen P0 and liquidity

Daily cross-sectional rank regressions control for:

- frozen P0 percentile score
- same-day ADV percentile

The strongest additional coefficients are:

| Feature | Mean incremental coefficient | t-stat |
|---|---:|---:|
| 60-day positive-day fraction | **+0.0288** | **2.95** |
| 60-day trend slope | **+0.0229** | **2.19** |
| MA20 / MA60 | **+0.0222** | **2.07** |
| signed 60-day Efficiency Ratio | +0.0178 | 1.79 |
| fraction above MA20 | +0.0166 | 1.69 |
| 60-day trend strength | +0.0165 | 1.63 |

These individual results are exploratory because they are inspected within the audit test sample. They should not be promoted directly into production without a new forward period.

## Trend smoothness result

Regression R2, used as a direct “smoothness/stability” measure, is weak:

- 20-day R2 standalone IC: about +0.0048
- 60-day R2 standalone IC: about -0.0038
- partial coefficients are also weak

Therefore the data does **not** support a simple hypothesis that “smoother historical trends are better next-day predictors.”

A more useful stability concept appears to be:

> **directional persistence across many days**, rather than geometric smoothness around a fitted line.

The 60-day positive-day fraction and signed Efficiency Ratio fit this interpretation better than R2.

## Current interpretation

The result separates three effects:

### 1. Short-term distance / overextension

Large positive close-to-MA5 distance is associated with weaker next-day relative performance.

This is consistent with short-horizon reversal.

### 2. Long-horizon direction

Persistent positive direction over roughly 60 days retains some incremental association after controlling the P0 score.

### 3. Generic MA crossover block

Combining many MA/trend variables into one model does not improve the frozen P0 and can overfit badly.

So the useful hypothesis is narrower than “moving averages work”:

> **Short-term overextension may mean-revert, while persistent longer-horizon direction may condition the next-day signal.**

## Next clean test

Do not tune another large trend block on the same test sample.

For the next untouched forward period, preregister a compact 3-feature trend conditioner:

1. 60-day positive-day fraction
2. MA20 / MA60
3. close / MA5 distance

Optionally keep signed 60-day Efficiency Ratio as a fourth feature.

Then compare:

```text
Frozen P0
vs
Frozen P0 + compact trend conditioner
```

The feature signs should be frozen before seeing the new period:

- 60-day persistence: positive
- MA20 / MA60: positive
- MA5 overextension: negative

## Decision

- Do **not** add a broad moving-average/trend-quality block to P0 now.
- Preserve the negative result.
- Promote only the compact long-persistence / short-overextension hypothesis to a future holdout test.
- Treat R2-style trend smoothness as low priority unless new evidence appears.

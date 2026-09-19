# Compact trend conditioner — true forward check through 2026-09-16 data

## Purpose

This is the first test performed **after** the compact trend hypothesis was frozen from the prior moving-average/trend-quality audit.

The frozen signs were:

1. 60-day positive-day fraction: positive
2. MA20 / MA60: positive
3. close / MA5 distance: negative (short-term overextension reversal)

No parameter, horizon or sign was changed after seeing the forward period.

## Forward data

The private Standard incremental update available on 2026-09-19 contains:

- complete August 2026 daily bars through 2026-08-31
- September daily bars through 2026-09-16
- matching historical security master
- TOPIX and market calendar updates

The prior research snapshot ended 2026-08-25.

Therefore the clean signal-date forward window is:

- **2026-08-26 through 2026-09-15**
- 15 trading days
- next-day target available through 2026-09-16
- median eligible cross-section: about 2,746 stocks/day
- same provisional ordinary-equity and ADV20 >= JPY 10m universe rules as P0

Target:

`next-day open -> close return`

## Frozen compact score

Each feature is ranked cross-sectionally on the signal date.

```text
compact_trend_score
  = rank(60d positive-day fraction)
  + rank(MA20 / MA60)
  - rank(close / MA5 distance)
```

The final score is ranked again cross-sectionally.

## True-forward result

| Signal | Mean daily rank IC | Positive IC days | t-stat |
|---|---:|---:|---:|
| 60-day positive-day fraction | +0.0019 | 53.3% | 0.08 |
| MA20 / MA60 | -0.0121 | 53.3% | -0.41 |
| short-term MA5 overextension reversal | **+0.1001** | **93.3%** | **3.37** |
| frozen 3-feature compact score | **+0.0452** | 66.7% | 1.88 |
| simple prior-day reversal | +0.0702 | 66.7% | 2.29 |

The compact score is positive, but its forward performance is almost entirely carried by the short-term MA5-overextension reversal component.

The two long-horizon continuation components that appeared interesting in the prior 120-stock audit do **not** confirm in this short forward window.

## Portfolio diagnostic

For the frozen compact score:

- top-decile minus same-day eligible-universe mean: **+15.6 bp/day gross**
- top-decile minus bottom-decile: **+31.4 bp/day gross**

Subperiod description:

| Period | Days | Compact trend IC | Simple reversal IC | Top-decile excess |
|---|---:|---:|---:|---:|
| Aug 26-Aug 31 | 4 | +0.0684 | +0.0354 | +10.4 bp/day |
| Sep 1-Sep 15 | 11 | +0.0368 | +0.0829 | +17.5 bp/day |

This is only 15 days and must not be annualized or treated as a stable net-alpha estimate.

## Is MA5 overextension just one-day reversal?

No, not completely.

Cross-sectionally, the rank correlation between:

- prior-day reversal `-ret1`
- MA5 overextension reversal `-close/MA5 distance`

averages about **0.66**.

A daily cross-sectional rank regression was run:

```text
next-day return rank
  ~ prior-day reversal rank
  + MA5-overextension-reversal rank
  + ADV rank
```

Across the 15 forward days:

- mean incremental MA5-overextension coefficient: **+0.0824**
- time-series t-stat of that coefficient: **2.63**
- mean incremental one-day-reversal coefficient: +0.0133
- corresponding t-stat: 0.42

This suggests the MA5-distance signal contains information beyond the previous day's return in this forward slice.

Top-decile gross excess:

- MA5-overextension reversal: **+32.1 bp/day**
- simple one-day reversal: +32.6 bp/day

Again, this is a very short window and gross of implementation costs.

## Interpretation

The prior audit suggested two ideas:

1. short-horizon overextension reverses;
2. longer-horizon directional persistence may continue.

The true-forward check supports the first idea and does **not** currently support the second.

So the updated evidence is narrower:

> **Distance above/below a fast moving average appears to capture a multi-day short-term overextension state that can predict next-day relative reversal beyond the previous day's return.**

The evidence does not currently justify adding generic MA crossover or long-trend-persistence blocks.

## Decision

- Keep broad moving-average/trend blocks rejected.
- Do not promote 60-day positive-day fraction or MA20/MA60 based on this forward period.
- Keep `-close/MA5 distance` as a candidate **short-term overextension state**.
- Do not retune the MA window using this forward period.
- Freeze MA5 as the next-forward candidate rather than searching MA3/MA7/MA10 now.
- The next untouched update after 2026-09-16 should test whether the MA5 effect persists.

## Limitation

This specific forward check tests the compact trend block directly against the next-day cross section and a simple one-day reversal baseline.

The exact frozen LightGBM P0 scorer was not persisted as a model artifact, so a clean post-2026-08-25 incremental `P0 vs P0 + MA5` comparison was not reconstructed in this pass. Re-training the full P0 merely to recreate the scorer would consume the previously untouched forward period if model choices were changed, so this report keeps the test narrow and auditable.

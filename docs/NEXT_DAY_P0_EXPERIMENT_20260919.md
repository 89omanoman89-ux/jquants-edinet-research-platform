# Next-day P0 experiment — 2026-09-19

## Status

**Prototype result: predictive signal is present out of sample, but executable net alpha is not yet established.**

This experiment uses only the private J-Quants Standard 10-year data already held by the user. No licensed raw data is committed to Git.

## Source and timing

Private source revision:

- source revision: `20260825T103719Z`
- raw entitlement period: 2016-08-01 through 2026-08-25
- source datasets used: daily equity bars, historical equity master, TOPIX daily
- financial summary exists but is **not used** in this P0 experiment
- incremental files after the base revision are not used in the formal P0 test below

The bulk daily-bars CSV has raw OHLC/volume/value plus `AdjFactor`; adjusted OHLC is not present in the bulk CSV. Daily returns are therefore adjusted for split/consolidation using the factor recorded on the new-scale date.

## Decision / execution contract

Main tradable experiment:

```text
information through close(t)
    -> compute score after close
    -> enter at open(t+1)
    -> exit at close(t+1)
```

Primary target:

```text
intraday_target(t+1) = Close(t+1) / Open(t+1) - 1
```

A separate `close(t) -> open(t+1)` target was also modeled as a predictability diagnostic. It is **not treated as executable alpha**, because the final close/volume/value features are only known after the period starts.

## Historical universe

Each date uses the contemporaneous `equities_master` row.

Provisional ordinary-equity universe:

- `ProdCat = 011`
- exclude `その他`
- exclude `TOKYO PRO MARKET`
- include historical TSE1/TSE2/Mothers/JASDAQ and current Prime/Standard/Growth as they existed at each date
- require valid OHLC, positive volume/value
- model floor: 20-day average trading value >= JPY 10 million

Typical modeled cross-section:

- 2017: median 2,653 names/day
- 2023: median 2,894 names/day
- 2024: median 2,876 names/day
- 2025: median 2,907 names/day
- 2026 through Aug-25: median 2,840 names/day

A separate liquid-universe diagnostic uses ADV20 >= JPY 50 million and has roughly 2,000 names/day.

## Data split

No random time split.

- train: 2017-2022
- validation: 2023
- untouched OOS: 2024-2026-08-25

Linear models use all 3,951,161 training observations.

The first LightGBM prototype uses a deterministic random 30% sample of training observations:

- train rows: 1,185,924
- features: 18
- validation: full 2023 sample
- early-stopping best iteration: 125

The 2024-2026 rows are never used for fitting or early stopping.

## P0 feature set

### Price path

- prior 1-day return
- same-day overnight return
- same-day intraday return
- trailing 5/20/60-day return
- market-adjusted 1-day return
- 17-sector-adjusted 1-day return

### Trading activity / liquidity / risk

- trading-value ratio versus 20-day average
- volume ratio versus 20-day average
- 20-day Amihud-style illiquidity proxy
- market capitalization log
- 20-day realized volatility
- daily high-low range
- close location within daily range

### Market state

- TOPIX daily return
- cross-sectional positive-return breadth
- cross-sectional return dispersion

No financial statement, TDnet text, U.S. market, FX, analyst, or tick feature is included yet.

## OOS result: hypothesis integration

Mean daily Spearman rank IC for next-day `open -> close`, ADV20 >= JPY 10m:

| Model | 2024 | 2025 | 2026 |
|---|---:|---:|---:|
| prior-day reversal only | 0.0284 | 0.0266 | 0.0148 |
| price-path block | 0.0328 | 0.0303 | 0.0198 |
| price + activity/liquidity | 0.0597 | 0.0624 | 0.0458 |
| full linear integrated | 0.0600 | 0.0631 | 0.0460 |
| LightGBM integrated | **0.0686** | **0.0721** | 0.0429 |

Day-weighted 2024-2026 mean IC:

| Model | Mean IC |
|---|---:|
| prior-day reversal only | 0.0244 |
| price-path block | 0.0287 |
| price + activity/liquidity | 0.0574 |
| full linear integrated | 0.0578 |
| LightGBM integrated | **0.0637** |

This is the central result of the prototype: integrating multiple information blocks materially improves cross-sectional predictability relative to a single reversal hypothesis.

However, the gain is **not evenly distributed across all hypotheses**. Most of the linear improvement arrives when trading-activity/liquidity/risk variables are added. Adding common market-state variables on top of that gives only a small further linear increment.

## Liquid-universe robustness

For ADV20 >= JPY 50m:

| Model | Day-weighted OOS mean IC 2024-2026 |
|---|---:|
| full linear | 0.0589 |
| LightGBM | 0.0602 |

The result therefore does not disappear when the lowest-liquidity portion of the universe is removed.

## LightGBM OOS detail

ADV20 >= JPY 10m:

| Year | Mean IC | IC-positive days | Top-decile excess, gross | Top-bottom spread, gross |
|---|---:|---:|---:|---:|
| 2024 | 0.0686 | 73.5% | +8.46 bp/day | +31.08 bp/day |
| 2025 | 0.0721 | 75.7% | +7.73 bp/day | +32.90 bp/day |
| 2026 | 0.0429 | 62.8% | +3.13 bp/day | +19.70 bp/day |

ADV20 >= JPY 50m:

| Year | Mean IC | Top-decile excess, gross | Top-bottom spread, gross |
|---|---:|---:|---:|
| 2024 | 0.0626 | +7.62 bp/day | +31.19 bp/day |
| 2025 | 0.0714 | +7.58 bp/day | +32.55 bp/day |
| 2026 | 0.0388 | +2.88 bp/day | +19.61 bp/day |

These are **gross cross-sectional returns, not implementable net returns**.

## Closing-auction regime split

The TSE closing-auction regime begins 2024-11-05.

LightGBM, ADV20 >= JPY 10m:

| Period | Days | Mean IC | Top-decile excess |
|---|---:|---:|---:|
| 2024 through Nov-01 | 205 | 0.0682 | +7.86 bp/day |
| 2024 Nov-05 onward | 40 | 0.0709 | +11.52 bp/day |

There is no immediate collapse in the short post-change window. The post-change sample is small and must not be treated as proof of structural improvement.

## Cost stress

Because the experiment enters at the opening auction and exits at the close every day, turnover is inherently high.

Simple round-trip cost subtraction from top-decile gross excess:

| Period | Gross | after 2 bp | after 5 bp | after 10 bp |
|---|---:|---:|---:|---:|
| 2024 pre-closing-auction | 7.86 | 5.86 | 2.86 | -2.14 |
| 2024 post-closing-auction | 11.52 | 9.52 | 6.52 | 1.52 |
| 2025 | 7.73 | 5.73 | 2.73 | -2.27 |
| 2026 | 3.13 | 1.13 | -1.87 | -6.87 |

This is only a stress test. It is not a transaction-cost estimate.

The strategy is **not robust to a blanket 10 bp round trip**, and 2026 is not robust to 5 bp.

## Concentration test

The signal is not dramatically stronger only in a handful of names.

Gross top-selection excess, ADV20 >= JPY 10m:

| Year | Top 10% | Top 5% | Top 2% | Top 1% |
|---|---:|---:|---:|---:|
| 2024 | 8.46 | 9.41 | 9.73 | 9.17 |
| 2025 | 7.73 | 7.73 | 8.06 | 7.95 |
| 2026 | 3.13 | 2.87 | 4.96 | 3.85 |

The predictive ranking is therefore relatively broad rather than being generated solely by the top few stocks.

## 2026 decay

Monthly mean IC in 2026:

| Month | Mean IC |
|---|---:|
| Jan | 0.0734 |
| Feb | 0.0243 |
| Mar | 0.0496 |
| Apr | 0.0449 |
| May | 0.0401 |
| Jun | 0.0475 |
| Jul | 0.0438 |
| Aug through base cutoff | 0.0097 |

2026 remains positive overall but is clearly weaker than 2024-2025. August has only 15 evaluated days in the base revision. Do not infer permanent decay yet.

## First LightGBM gain importance

The first prototype's gain ranking:

1. sector-adjusted prior-day return
2. log market capitalization
3. 20-day volatility
4. cross-sectional return dispersion
5. market breadth
6. TOPIX prior-day return
7. 5-day return
8. same-day overnight return
9. daily high-low range
10. 60-day return

Importance is not causal evidence and correlated variables can substitute for one another.

## Important interpretation

### What the test supports

- A broad J-Quants-only cross-sectional model can rank roughly 2,800-2,900 Japanese stocks per day.
- A single reversal hypothesis contains information, but integrating other blocks improves OOS rank predictability materially.
- The effect survives a JPY 50m ADV filter.
- A nonlinear model improves 2024-2025 relative to the linear model.

### What the test does not support yet

- Net profitability after real spread, fees, opening/closing impact and capacity.
- A claim that all hypotheses add independent information.
- Stability beyond the observed 2026 weakening.
- Use of the non-executable `close -> next open` diagnostic as trading alpha.
- Causal interpretation of feature importance.
- Live deployment.

## Next falsification steps

1. **Real execution audit**
   - use the private 120-stock trade-tick pack from 2024-09 onward
   - measure opening/closing auction traded volume and realistic participation
   - the pack contains trades, not quotes/order book, so it cannot directly measure bid-ask spread

2. **2026 base-cutoff extension**
   - append incremental Standard daily updates after 2026-08-25
   - test whether August weakness continues

3. **Walk-forward retraining**
   - current LightGBM is trained only on 2017-2022 and selected on 2023
   - compare frozen model against annual expanding-window refits without touching future periods

4. **Feature-block falsification**
   - isolate trading-value ratio vs volume ratio
   - test whether their combination is mostly a disguised price/VWAP-relative signal
   - remove market-cap and volatility to measure dependency

5. **Portfolio construction**
   - long-only top-N and sector-neutral top-N
   - turnover-aware score smoothing
   - capacity by ADV participation
   - avoid treating the short leg as freely borrowable

6. **Only after P0 survives**
   - add financial-summary/forecast revisions
   - then TDnet
   - then external U.S./sector data
   - order-book data remains a separate P3 layer

## Current conclusion

The experiment is strong enough to justify continuing.

The key result is not that “LightGBM predicts stocks.” The result is narrower:

> In a timestamp-clean, broad Japanese-stock universe, multiple weak daily price/activity/liquidity hypotheses produce materially higher untouched OOS cross-sectional rank IC than a simple reversal baseline.

The current bottleneck has shifted from **whether any predictive signal exists** to **whether the signal survives realistic execution costs and 2026 regime decay**.

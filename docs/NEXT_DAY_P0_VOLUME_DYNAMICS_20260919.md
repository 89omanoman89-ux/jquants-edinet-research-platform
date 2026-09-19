# P0 volume-dynamics ablation — 2026-09-19

## Question

Does modeling **changes in volume / trading value**, beyond the existing P0 level-relative activity features, materially improve next-day Japanese equity ranking?

The base P0 already contains:

- `val_ratio20 = log(Va / ADV20)`
- `vol_ratio20 = log(Vo / mean20(Vo))`
- Amihud-style illiquidity
- market cap
- volatility / range / close-location
- price-path and market / sector controls

This follow-up adds explicit *change-shape* variables.

## Added daily features

### Short-horizon change

- `vol_chg1 = log(Vo_t / Vo_{t-1})`
- `val_chg1 = log(Va_t / Va_{t-1})`
- `vol_ratio5 = log(Vo_t / mean5(Vo))`
- `val_ratio5 = log(Va_t / mean5(Va))`

### Relative shock / acceleration

- `vol_z20`
- `val_z20`
- `vol_accel = vol_ratio5 - vol_ratio20`
- `val_accel = val_ratio5 - val_ratio20`
- turnover level / 20-day z-score

### Price × volume interaction

- `ret1 × vol_z20`
- `abs(ret1) × vol_z20`
- positive-return × volume shock
- negative-return × volume shock
- intraday-return × volume shock
- return × trading-value shock
- daily-range × volume shock
- close-location × volume shock

Raw-share volume-change features are invalidated for 20 trading days around split / consolidation events so mechanical share-count jumps are not treated as alpha.

## Experimental contract

Unchanged from the main P0 experiment:

- train: 2017-2022
- validation / early stopping: 2023
- untouched OOS: 2024-2026-08-25
- same historical common-stock universe
- same ADV20 >= JPY 10m eligibility floor
- same target: next-day open -> close cross-sectional rank
- same LightGBM hyperparameters
- deterministic 30% training sample: 1,185,924 rows
- full validation and full OOS evaluation

The base 18 columns in the expanded dataset are byte-for-byte identical to the original P0 model arrays.

## Main result

### ADV20 >= JPY 10m

Day-weighted OOS 2024-2026:

| Model | Mean IC | Top-10% excess | Delta IC vs base | Delta top-10% |
|---|---:|---:|---:|---:|
| base P0 | 0.063718 | 6.894 bp/day | — | — |
| + all volume dynamics | 0.064616 | 6.675 bp/day | +0.000898 | -0.219 bp/day |
| + full dynamics + interactions | 0.064525 | 6.625 bp/day | +0.000808 | -0.269 bp/day |
| + top-6 change variables | 0.064565 | 6.972 bp/day | +0.000847 | +0.078 bp/day |
| + trading-value-change group | **0.064670** | **6.974 bp/day** | **+0.000953** | **+0.080 bp/day** |
| + share-volume-change group | 0.063737 | 6.513 bp/day | +0.000019 | -0.381 bp/day |

The improvement in rank IC is real but small. It does **not** translate into a robust improvement in top-decile gross return.

## Pre-specified all-dynamics model by year

This is the cleanest incremental test because the full daily volume-dynamics block was specified before its OOS result was inspected.

| Year | Base IC | + volume dynamics IC | Delta |
|---|---:|---:|---:|
| 2024 | 0.068638 | 0.067848 | -0.000790 |
| 2025 | 0.072110 | 0.074477 | +0.002367 |
| 2026 | 0.042917 | 0.044180 | +0.001263 |

Paired daily IC delta over all 644 OOS days:

- mean delta: **+0.000898**
- paired t-stat: **1.68**
- positive-delta day rate: **51.1%**

Top-decile excess delta:

- mean delta: **-0.219 bp/day**
- paired t-stat: **-0.67**

This is not strong enough to claim a robust independent alpha block.

## Liquid universe

ADV20 >= JPY 50m, day-weighted 2024-2026:

| Model | Mean IC | Top-10% excess |
|---|---:|---:|
| base P0 | 0.061186 | 6.454 bp/day |
| + all volume dynamics | 0.061653 | 6.439 bp/day |
| + trading-value-change group | 0.061799 | 6.436 bp/day |

Again, incremental IC is modest and top-decile payoff is essentially unchanged.

## Which new features were actually used?

In the full volume-dynamics model, the highest-gain new features were:

1. `val_accel`
2. `vol_accel`
3. `val_chg1`
4. `val_ratio5`
5. `vol_ratio5`
6. `vol_chg1`

The 20-day volume/value z-scores and turnover z-score had much smaller gain. Turnover level had effectively zero incremental gain.

This suggests that **changes in the activity baseline** matter more than another standardized level measure.

## Exploratory focused groups

After inspecting training/validation feature importance, smaller follow-up groups were tested.

These are exploratory and should not be treated as a fresh untouched-holdout discovery.

### Trading-value change

Features:

- `val_accel`
- `val_chg1`
- `val_ratio5`

OOS 2024-2026:

- weighted IC: **0.064670**
- delta vs base: **+0.000953**
- paired IC-delta t-stat: **1.76**
- top-decile delta: **+0.080 bp/day**
- top-decile-delta t-stat: **0.27**

### Share-volume change

Features:

- `vol_accel`
- `vol_chg1`
- `vol_ratio5`

OOS weighted IC: **0.063737**, essentially identical to base, while top-decile excess is lower.

Trading-value changes therefore look more useful than raw share-volume changes in this experiment.

## Interpretation

The correct conclusion is **not** "volume change does not matter."

The narrower result is:

> The original P0's 20-day relative trading activity and liquidity variables already capture most of the daily total-volume information. Additional one-day / five-day / acceleration features add only a small amount of independent rank information.

The fact that trading-value changes outperform raw share-volume changes is economically sensible as a working hypothesis: trading value jointly reflects quantity and price scale and is less mechanically distorted by changes in shares-per-unit than raw volume.

This interpretation still requires forward confirmation.

## Price × volume interaction result

Adding explicit interaction features does not improve the model beyond the simpler volume-dynamics block.

Therefore the current LightGBM already appears able to learn much of the useful nonlinearity from the underlying price and activity features without hand-building many cross terms.

This is a useful negative result: avoid expanding the daily feature set merely because an interaction has an intuitive story.

## Next volume experiment

The next high-value information is probably **when during the day volume occurs**, not another transformation of total daily volume.

The private 120-stock trade-tick sample can support features such as:

- first 5 / 30 minute share of daily volume
- final 5 / 30 minute share
- closing-auction share
- morning vs afternoon volume imbalance
- change in intraday volume-curve shape versus each stock's history
- number of trades / average trade size changes
- late-day volume shock conditional on return direction
- price move during high-volume intervals

These should be treated as a separate P3 microstructure/activity-timing block and tested only with information available by the decision cutoff.

## Current decision

Keep the base P0 activity block.

Do **not** promote the expanded total-volume dynamics block as a major independent alpha source yet.

If a compact daily addition is retained for future forward testing, the most defensible candidates are:

- trading-value acceleration
- one-day trading-value change
- five-day-relative trading value

The stronger next research direction is intraday **volume timing / volume curve**, because total daily volume transformations now show clear diminishing returns.

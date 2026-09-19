# P0 intraday volume-curve / closing-price-path audit — 2026-09-19

## Question

Does **when volume trades during the signal day** add predictive information beyond the existing daily P0 model?

This experiment uses only the private 120-stock executed-trade sample and the already frozen full-universe P0 score. No raw licensed tick rows are committed to Git.

## Sample and split

The tick sample was selected in 2024-08 without return-based selection:

- 120 common stocks
- 40 low / 40 middle / 40 high tick-activity names
- sector round-robin selection
- no survivor filter
- evaluation data begins 2024-09

For this experiment:

- 2024-09: rolling-history burn-in
- train: 2024-10 through 2025-06
- validation / early stopping: 2025-07 through 2025-09
- untouched test for this experiment: 2025-10 through 2026-07
- test days: 203
- tick names available: 119

The frozen P0 score is calculated in the full approximately 2,800-2,900-stock daily universe first, then joined onto the 120-stock tick sample.

## Intraday features

### Pure volume / notional timing

- opening-auction share of daily volume
- closing-auction share of daily volume
- first-5-minute share of daily volume
- final-5-minute/closing-window share of daily volume
- first-5-minute share of daily notional
- final-5-minute share of daily notional
- open+close auction concentration
- first+last 5-minute concentration
- closing-vs-opening volume-share log ratio
- final-5-vs-first-5 volume-share log ratio

For each shape feature, also test:

- 1-day change
- z-score against the stock's prior 20 observed trading days

The prior-20 baseline is shifted by one day, so the current value is not used to build its own historical mean/std.

### Intraday price-path controls

Two additional end-of-day price-formation variables are kept separate from volume timing:

- first-5-minute VWAP move versus the opening price
- final-5-minute VWAP to closing-price move

These are known only after the signal-day close and therefore are eligible only for next-day trading.

## Test architecture

A secondary LightGBM is trained on the tick sample with the frozen full-universe P0 percentile score as a monotonic-positive input.

Models:

1. raw frozen P0 score
2. P0 + pure volume-timing features
3. P0 + intraday edge-price-path features only
4. P0 + both volume timing and edge price path

Primary metric: daily Spearman rank IC versus next-day open-to-close return.

## Main result

Untouched 2025-10 through 2026-07:

| Model | Mean daily IC | IC-positive days | Top-10% excess inside tick sample |
|---|---:|---:|---:|
| frozen P0 | 0.05161 | 62.6% | +1.25 bp/day |
| + pure volume timing | 0.04325 | 62.6% | +4.20 bp/day |
| + edge price path only | **0.06383** | 69.0% | **+7.50 bp/day** |
| + full timing block | 0.06189 | **69.5%** | +5.40 bp/day |

### Paired daily difference vs frozen P0

Pure volume timing:

- delta IC: **-0.00836**
- paired t-stat: -1.41

Edge price path only:

- delta IC: **+0.01222**
- paired t-stat: 1.50
- top-decile delta: +6.26 bp/day
- top-decile delta t-stat: 1.35

Full timing block:

- delta IC: **+0.01028**
- paired t-stat: 1.33

Therefore the first conclusion is:

> The apparent gain from the broader intraday timing block is **not primarily caused by volume timing**. It is driven mainly by late-day price-formation information.

The test sample is small enough that the model-level improvement should be treated as promising, not established.

## Direct volume-curve evidence

Pure volume timing is not entirely uninformative.

Direct OOS daily rank IC of individual timing variables:

- closing-auction share: about +0.039
- final-5-minute share: about +0.035
- total open+close auction share: about +0.038
- first-5-minute share: about -0.008

In daily cross-sectional regressions controlling for:

- frozen P0 score percentile
- same-day ADV percentile

the volume-timing coefficient is:

| Feature | Mean incremental coefficient | Time-series t-stat |
|---|---:|---:|
| closing-auction share rank | **+0.03775** | **2.34** |
| total auction-share rank | **+0.03024** | **2.19** |
| final-5-minute share rank | +0.02794 | 1.87 |
| first+last-5-minute concentration rank | +0.01446 | 1.18 |

This is useful but narrower than a standalone model improvement.

Interpretation:

> Stocks with unusually large end-of-day/auction participation tend to have somewhat stronger next-day open-to-close returns even after controlling for the existing P0 score and liquidity in this 120-stock sample.

This is an association, not yet a deployable alpha.

## Compact linear integration check

A strongly regularized Ridge model using only one timing-rank variable plus P0 score and ADV gives:

- P0-like linear baseline test IC: approximately 0.0514
- + closing-auction-share rank: approximately 0.0535
- + final-5-minute-share rank: approximately 0.0532

So pure volume timing may provide a **small** incremental signal when integrated conservatively, even though the large nonlinear volume-timing block overfits.

## Late-day price-path finding

The strongest new input in the nonlinear model is:

- final-5-minute VWAP relative to the closing price

Its direct next-day rank IC is negative (about -0.033): a close that finishes above the preceding final-5-minute VWAP tends to be followed by weaker next-day intraday relative performance, consistent with a short-horizon end-of-day reversal/price-pressure interpretation.

Other high-gain inputs include:

- change in first-5-minute price move
- change in final-5-minute-to-close move
- final-5-minute notional-share change
- closing-auction share
- closing-auction-share surprise versus prior 20 days

This finding should be connected back to the short-term reversal / market-microstructure literature already catalogued in the research repo.

## Subperiod check

Mean rank IC:

| Model | 2025 Q4 | 2026 Jan-Jul |
|---|---:|---:|
| frozen P0 | 0.0600 | 0.0479 |
| + pure volume timing | 0.0488 | 0.0408 |
| + edge price path only | **0.0689** | **0.0616** |
| + full timing block | 0.0682 | 0.0591 |

The late-day price-path gain is present in both subperiods, but the sample remains too small to promote it directly into production.

## Decision

### Keep researching

- closing-auction share
- final-5-minute volume share
- change/surprise in those shares relative to each stock's own history
- final-5-minute VWAP to closing-price move

### Do not do

- do not add dozens of raw volume-curve transformations to the production P0 model
- do not claim pure intraday volume timing improves the model based on the current nonlinear result
- do not treat the 120-stock sample as a full-market holdout

## Best next test

The next clean test should use a larger tick universe or future tick data and preregister a **compact 3-4 feature block**:

1. closing-auction share rank
2. final-5-minute volume-share surprise vs prior 20 days
3. final-5-minute VWAP -> close move
4. change in that late-day price move vs prior day/history

Then compare:

```text
Frozen P0
vs
Frozen P0 + compact end-of-day microstructure block
```

without reopening feature selection.

## Current conclusion

The high-value microstructure signal appears to be more specific than "volume timing":

> **End-of-day participation and final-minutes price formation contain some incremental next-day information, but broad intraday volume-curve engineering adds too much noise.**

The most promising next hypothesis is therefore a compact **closing-auction / late-day price-pressure block**, not a large generic intraday-volume block.

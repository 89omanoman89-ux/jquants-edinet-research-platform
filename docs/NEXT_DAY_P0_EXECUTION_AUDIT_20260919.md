# Next-day P0 execution-capacity audit — 2026-09-19

## Status

**The P0 signal remains statistically visible in the independent 120-stock tick sample, but pure open/close-auction execution is capacity-constrained outside the high-activity tier.**

This audit uses the user's private J-Quants trade-tick pack. No licensed raw trades are committed to GitHub.

## Tick sample design

The private tick pack was constructed independently from the P0 return model:

- selection month: 2024-08
- 120 common stocks
- 3 tick-activity tiers: low / middle / high
- 40 stocks per tier
- sector round-robin within each tier
- deterministic SHA256 ranking within sector
- no return-based selection
- no future-survivor filter
- evaluation starts 2024-09

The continuous monthly evaluation set used here is:

- 2024-09 through 2026-07
- 23 monthly Parquet files
- 47,621,494 trade records
- 55,029 stock-days before P0 eligibility filtering

The pack contains executed trades only:

```text
Date
Code
Time
SessionDistinction
Price
TradingVolume
TransactionId
SourceRow
```

It does **not** contain quotes, order-book depth, queue position, or bid-ask spread.

## P0 score reconstruction

The frozen P0 LightGBM from the earlier experiment is reused without retraining.

For each signal date:

1. compute the full eligible Japanese-stock cross-section;
2. predict next-day open-to-close rank score;
3. rank across the full approximately 2,800-2,900-stock universe;
4. retain only the 120 tick-sample stocks afterward.

This avoids ranking only inside the 120-stock sample.

Joined observations:

- P0-eligible tick stock-days: 43,728
- unique tick stocks joined: 119
- trading days: 466
- full-universe top-10% observations in tick sample: 4,968

## Auction definitions

### Opening execution

Opening auction proxy:

```text
the first executed trade for each stock/date
```

This is appropriate for a pre-open market-on-open order. Delayed openings remain delayed rather than being forced to 09:00.

### Closing execution

Scheduled closing-auction trades are executions exactly at:

- before 2024-11-05: `15:00:00.000000`
- from 2024-11-05: `15:30:00.000000`

If no trade exists at the scheduled closing time, closing-auction capacity is treated as zero.

## Price tie-out

For stock-days with a scheduled closing-auction print:

- observations: 43,120
- median absolute difference between tick open→close return and J-Quants daily target: **0.000185 bp**
- 95th percentile absolute difference: **0.000538 bp**
- within 0.01 bp: **99.9165%**

Therefore the first trade and scheduled closing-auction print reproduce the daily open/close target essentially exactly for the usable sample.

## Predictive validation inside the 120-stock tick sample

This is a small, stratified sample and is **not** a replacement for the full-universe OOS test.

Daily Spearman rank IC:

| Period | Mean IC | IC-positive days | IC t-stat |
|---|---:|---:|---:|
| 2024-09 onward | 0.0732 | 70.7% | 4.20 |
| 2025 | 0.0518 | 65.8% | 5.44 |
| 2026 through Jul | 0.0506 | 61.0% | 3.42 |
| All | **0.0552** | **65.2%** | **7.51** |

Full-universe-top-10% signal minus same-day eligible tick-sample mean:

| Period | Daily excess | t-stat |
|---|---:|---:|
| 2024-09 onward | +14.95 bp | 3.78 |
| 2025 | +1.12 bp | 0.34 |
| 2026 through Jul | +3.09 bp | 0.54 |
| All | +4.15 bp | 1.63 |

Interpretation:

- the score ordering remains predictive in the tick sample;
- the small sample's top-decile payoff is much less stable than the full-universe IC;
- 2024 is strong, 2025-2026 are not statistically convincing in this 120-stock sample.

## Pure-auction capacity

For P0 full-universe top-10% signals observed in the tick sample:

- observations: 4,968
- scheduled close-auction availability: **99.0%**
- delayed opening >60 seconds: **5.8%**
- median opening-auction notional: **JPY 13.65m**
- median closing-auction notional: **JPY 59.79m**
- median round-trip auction bottleneck: **JPY 11.97m**

Median share of daily traded notional:

- opening auction: **4.41%**
- closing auction: **17.12%**

The opening auction is therefore the main capacity bottleneck.

### Participation-cap interpretation

Median per-stock capacity using only opening/closing auctions:

| Auction participation cap | Median capital/name |
|---|---:|
| 1% | about JPY 0.12m |
| 2% | about JPY 0.24m |
| 5% | about JPY 0.60m |
| 10% | about JPY 1.20m |

These are capacity figures only. They are not impact-free fill guarantees.

## Capacity by tick-activity tier

P0 top-10% signals are tilted toward the higher-activity sample:

- high: 53.1%
- middle: 32.2%
- low: 14.7%

Median round-trip auction notional:

| Tier | Median ADV20 | Auction bottleneck |
|---|---:|---:|
| high | JPY 1.38bn | **JPY 52.69m** |
| middle | JPY 64.5m | **JPY 2.43m** |
| low | JPY 25.3m | **JPY 0.96m** |

At a 5% auction-participation cap, the corresponding median per-stock capital is roughly:

- high: **JPY 2.63m**
- middle: **JPY 0.12m**
- low: **JPY 0.05m**

Thus a pure-auction implementation is economically plausible mainly in the high-activity tier unless allocations are very small.

## JPY 1m/name fill diagnostic

For P0 top-10% signals:

### Pure auction

| Participation cap | Full-fill rate | Median fill fraction |
|---|---:|---:|
| 1% | 17.4% | 12.2% |
| 5% | 42.5% | 60.9% |

High-activity tier only:

| Participation cap | Full-fill rate | Median fill fraction |
|---|---:|---:|
| 1% | 30.6% | 52.7% |
| 5% | **71.9%** | **100%** |

Middle/low tiers remain strongly capacity constrained.

## Five-minute execution windows

To test whether capacity can be increased without relying only on the opening print:

### Entry window

- opening auction
- plus first five minutes after the stock's actual first trade

### Exit window

Before 2024-11-05:

- last five continuous minutes through 15:00, including close print

From 2024-11-05:

- 15:20-15:25 continuous trading
- plus 15:30 closing auction

This respects the 15:25-15:30 pre-closing no-trade period.

For P0 top-10% signals:

- median pure-auction round-trip capacity: JPY 12.19m
- median five-minute-window round-trip capacity: **JPY 22.15m**
- median multiplier: **1.67x**

At JPY 1m/name:

| Mode | Participation | Full-fill rate | Median fill |
|---|---:|---:|---:|
| auction | 1% | 17.4% | 12.2% |
| five-minute window | 1% | **28.8%** | **22.2%** |
| auction | 5% | 42.5% | 60.9% |
| five-minute window | 5% | **51.7%** | **100%** |

High-activity tier, JPY 1m/name:

- five-minute window at 1%: 50.3% fully filled
- five-minute window at 5%: **84.6% fully filled**

## Price effect of spreading execution

Using full observed five-minute VWAPs as a diagnostic:

For top-10% signals:

- mean five-minute-return minus pure-auction-return: **+0.06 bp**
- median difference: **0.0 bp**
- 10th percentile: about **-22.7 bp**
- 90th percentile: about **+21.9 bp**

By year:

| Year | Mean five-minute minus auction return |
|---|---:|
| 2024 Sep-Dec | -1.72 bp |
| 2025 | -0.15 bp |
| 2026 Jan-Jul | +1.63 bp |

There is no stable average penalty in this sample, but the stock-day dispersion is large. Full-window VWAP is not a guaranteed executable price and must not be treated as an impact-free cost estimate.

## Closing-auction regime description

For top-10% sample observations:

| Period | Median close-auction notional | Median round-trip auction capacity |
|---|---:|---:|
| before 2024-11-05 | JPY 42.0m | JPY 8.32m |
| from 2024-11-05 | JPY 66.0m | JPY 12.69m |

This is descriptive only. The periods differ in calendar time and sample composition; it does not identify a causal effect of the new closing-auction mechanism.

## Tier-specific predictive/economic read-through

### High activity

All-period:

- mean rank IC: **0.0415**
- top-10% daily excess vs same-tier sample: **+7.29 bp**
- excess t-stat: **1.97**
- strong execution capacity relative to other tiers

But 2026 high-tier IC falls to 0.0136 in this 120-stock sample, so high-liquidity implementation is not yet proven stable.

### Middle activity

- mean rank IC: 0.0580
- top-10% daily excess: +1.35 bp
- materially weaker capacity

### Low activity

- mean rank IC: 0.0677
- top-10% daily excess: -2.01 bp
- very low auction capacity

High statistical rank IC in low-activity names does not automatically translate into executable top-decile alpha.

## What this audit establishes

1. Tick executions tie out almost perfectly to the daily open/close labels.
2. The frozen P0 score remains positively rank-correlated with returns in the independent tick sample.
3. Opening-auction liquidity is the dominant capacity bottleneck.
4. Pure-auction execution at meaningful per-name capital is mainly feasible in the high-activity tier.
5. A five-minute execution window roughly doubles median capacity, but introduces stock-day price-path risk.
6. Execution constraints materially strengthen the case for a liquid/high-activity implementation rather than an all-stock equal-weight portfolio.

## What remains unresolved

This trade-only pack cannot directly measure:

- bid-ask spread
- displayed/hidden depth
- queue position
- order-book imbalance
- price impact caused by our own order
- market-order versus limit-order fill probabilities
- brokerage/exchange fees
- borrow costs

Therefore this is an **execution-capacity audit**, not a complete net-PnL backtest.

## Next experiment

The next implementation test should be:

1. impose a liquid/high-activity universe;
2. use turnover-aware score persistence rather than a full daily replacement;
3. compare pure open/close auctions versus five-minute execution windows;
4. explicitly penalize forecast changes that cause turnover;
5. reserve 2026 as the stress period;
6. only after that add financial/disclosure blocks.

The current strongest implementation hypothesis is:

> **The P0 cross-sectional signal may be tradable only after explicitly optimizing for liquidity, execution-window capacity, and turnover; the all-stock daily top-decile portfolio is not yet supported as executable net alpha.**

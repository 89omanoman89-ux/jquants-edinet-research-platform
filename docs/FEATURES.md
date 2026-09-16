# FEATURES

Gold層では、raw値そのものだけでなく研究用特徴量を生成できる設計にする。

すべての特徴量は、入力データ、source、計算定義、利用可能時点を追跡可能にする。

以下は候補であり、実装確定リストではない。

## Human Capital

- `revenue_per_employee`
- `operating_profit_per_employee`
- `employee_growth`
- `salary_growth`
- `revenue_growth_minus_employee_growth`

## Investment

- `rnd_to_sales`
- `rnd_growth`
- `capex_to_sales`
- `capex_to_assets`
- `capex_to_depreciation`
- `investment_acceleration`

## Balance Sheet

- `net_debt_to_equity`
- `goodwill_to_equity`
- `investment_securities_to_equity`
- `inventory_growth_minus_sales_growth`

## Ownership

- `top1_ownership`
- `top5_ownership`
- `top10_ownership`
- `ownership_concentration`
- `cross_shareholding_to_equity`
- `cross_shareholding_reduction`

## Segment

- `segment_hhi`
- `largest_segment_share`
- `fastest_segment_growth`
- `segment_profit_dispersion`
- `segment_growth_dispersion`

## Text / Disclosure change

将来候補:

- `risk_text_change`
- `mda_similarity`
- `new_risk_topics`
- `removed_risk_topics`

## Feature quality

各featureについて最低限以下を文書化する。

- definition
- input fields
- source priority
- formula
- unit
- observation timing
- missing policy
- winsorization等の後処理
- known limitations

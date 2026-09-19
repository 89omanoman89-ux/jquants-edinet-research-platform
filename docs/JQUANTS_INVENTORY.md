# JQUANTS_INVENTORY

> この文書は既存J-Quantsデータを実際に調査して更新する。推測で埋めない。

更新確認日: 2026-09-19

## 調査ルール

- 元データはread-only。
- 列名だけで意味を断定しない。
- 不明な列は可能な範囲でJ-Quants公式仕様と照合する。
- 実データと文書が矛盾する場合は差異を記録する。
- private契約データそのもの、Drive URL/ID、認証情報はGitへ追加しない。
- Gitにはschema、coverage、品質監査、再現手順だけを保存する。

## 確認済み private dataset

Google Drive上の本人専用J-Quants Standard 10-year packをread-onlyで確認した。

- base source revision: `20260825T103719Z`
- raw entitlement period: `2016-08-01` through `2026-08-25`
- source files: 572
- raw source bytes: 1,186,566,607 bytes
- raw `.csv.gz` files downloaded through base cutoff: 545
- normalized research DuckDB: 683,421,696 bytes
- normalized DuckDB SHA-256: `5ee1ac8eb269ea95aba9e4ab3687b3bbadbdacaabb0d0e9f2fe48fdfd4d93ac0`
- incremental Standard update bundle through `2026-09-16` も存在する
- API key / authentication material / signed URLs はpackに含まれない

private packは第三者へ再配布しない。研究コード・schema・結果と実データを分離する。

## Dataset coverage

| Dataset | 確認済みraw file構成 | Coverage | 直近確認schema | 翌日予測での主用途 |
|---|---:|---|---|---|
| `equities_bars_daily` | 120 monthly + 16 daily files in base revision | 2016-08 through 2026-08-25 base; incremental updates observed through 2026-09-16 | 13 columns | OHLCV、売買代金、調整係数、時価総額、短期return、liquidity |
| `equities_master` | 120 monthly + 16 daily files in base revision | same base period; incremental updates observed through 2026-09-16 | 14 columns | historical universe、市場区分、17/33業種、規模、信用区分、商品区分 |
| `fins_summary` | 120 monthly + 16 daily files in base revision | same base period; incremental updates observed through 2026-09-15 in inspected update bundle | 111 columns | 開示時刻、実績、会社予想、配当予想、revision/event features |
| `indices_topix_daily` | 120 monthly + 16 daily files in base revision | same base period; incremental updates observed through 2026-09-16 | 5 columns | market return、market-adjustment、regime state |
| `markets_calendar` | 1 raw data file in base revision | inspected calendar begins 2008-01-01 | 2 columns | trading-day alignment、next-session label |

base source-tree manifestで上記5 datasetすべてがentitlement PASSであることを確認した。

## File inventory / recent inspected snapshots

| File class | Format | Rows in inspected snapshot | Date | Securities / events | Key candidate | Notes |
|---|---|---:|---|---:|---|---|
| daily equity bars | csv.gz | 4,442 | 2026-09-08 | 4,442 codes | `Date, Code` | ETF等を含む全商品。OHLC null 242、MktCap null 710、ExRTは当該snapshotでは全null |
| equity master | csv.gz | 4,447 | 2026-09-16 | 4,447 codes | `Date, Code` | Prime 1,555 / Standard 1,561 / Growth 598 / Other 546 / TOKYO PRO 187 |
| financial summary | csv.gz | 11 | 2026-09-15 | 11 disclosures | `DiscDate, DiscTime, Code, DiscNo` | event-driven sparse daily file。111 columns |
| TOPIX daily | csv.gz | 1 | 2026-09-16 | 1 index row | `Date` | OHLC |
| market calendar | csv.gz | 7,305 | 2008 onward | n/a | `Date` | `HolDiv`あり |

上表のrow数・欠損は特定snapshotの観測値であり、全期間へ一般化しない。

## Schema inventory

### equities_bars_daily

直近実ファイルで確認したcolumns:

```text
Date
Code
O
H
L
C
UL
LL
Vo
Va
AdjFactor
MktCap
ExRT
```

実データ型の例:

- `Date`: date-like string
- `Code`: security code; stringとして読む
- `O/H/L/C`: numeric
- `Vo`: volume
- `Va`: trading value
- `AdjFactor`: adjustment factor
- `MktCap`: market capitalization field
- `UL/LL/ExRT`: 正式意味と研究利用条件を公式仕様で再確認するまで推測しない

### equities_master

直近実ファイルで確認したcolumns:

```text
Date
Code
CoName
CoNameEn
S17
S17Nm
S33
S33Nm
ScaleCat
Mkt
MktNm
Mrgn
MrgnNm
ProdCat
```

2026-09-16 snapshotで観測した市場区分:

- Prime: 1,555
- Standard: 1,561
- Growth: 598
- Other: 546
- TOKYO PRO MARKET: 187

`ProdCat=11` は同snapshotで3,894件。  
`Mkt in {Prime, Standard, Growth}` かつ `ProdCat=11` の観測は3,707件。

これは翌日予測universeの有力候補だが、`ProdCat=11` の正式な商品定義をJ-Quants仕様で確認してからproduction filterへ固定する。

同日のbarsと結合すると、この3,707候補のうち:

- bars matched: 3,707
- non-null OHLC: 3,677
- positive volume: 3,677
- MktCap available: 3,671

したがって約3,000銘柄規模のcross-sectional daily modelはデータ上実現可能。

### fins_summary

直近実ファイルで111 columnsを確認。主要な列群:

```text
DiscDate
DiscTime
Code
DiscNo
DocType
CurPerType
CurPerSt
CurPerEn
CurFYSt
CurFYEn
Sales
OP
OdP
NP
EPS
TA
Eq
BPS
CFO
CFI
CFF
CashEq
...
FSales2Q / FOP2Q / FOdP2Q / FNP2Q / FEPS2Q
FSales / FOP / FOdP / FNP / FEPS
NxF...
Div...
FDiv...
ROE
NCROE
```

重要点:

- `DiscDate` と `DiscTime` を保持している。
- after-close / before-close判定が可能。
- 会社予想・次期予想・配当予想revisionをPIT event featureへ変換できる。
- financial fieldsは欠損が非常に多いevent rowがあるため、0埋め禁止。
- `DocType` ごとに利用可能fieldを分ける。

### indices_topix_daily

```text
Date
O
H
L
C
```

market-adjusted return、market state、beta/residual return生成に使用可能。

### markets_calendar

```text
Date
HolDiv
```

翌営業日target生成では単純なcalendar-day shiftを使わず、当該calendarに基づくnext trading dayを使用する。

## P0 next-day model で作成可能なfeature

J-Quants private packだけで、少なくとも以下を構築可能。

### Price / return

- `ret_1d`
- `ret_2d`
- `ret_5d`
- `ret_20d`
- `ret_60d`
- `overnight_ret = O_t / C_{t-1} - 1`
- `intraday_ret = C_t / O_t - 1`
- TOPIX-adjusted return
- sector-relative return
- rolling high/low distance
- opening gap
- high-low range
- close location within daily range

### Volume / liquidity

- volume ratio vs 5/20/60d
- trading-value ratio
- turnover proxy
- ADV
- Amihud-style illiquidity
- zero/no-trade or missing-trade flag
- return × abnormal-volume interaction
- reversal × liquidity interaction

### Risk / state

- realized volatility
- downside volatility
- idiosyncratic-volatility proxy against TOPIX
- market volatility state
- cross-sectional dispersion
- market/sector breadth

### Cross-sectional metadata

- 17-sector
- 33-sector
- market section
- TOPIX size category
- margin classification
- market capitalization rank

### Disclosure / forecast

- before/after-close disclosure
- earnings / dividend forecast revision
- SUE-like numeric surprise where history permits
- management forecast change
- actual vs prior forecast deviation
- disclosure-count/event-density features

## Target definition

P0では少なくとも2 targetを別々に構築する。

```text
overnight_target[t+1] = O[t+1] / C[t] - 1
intraday_target[t+1]  = C[t+1] / O[t+1] - 1
```

`close_to_close` は補助targetとして保持するが、overnightとintradayを最初から混ぜない。

当日closeをfeatureに使うrunでは、当日close価格での約定を仮定しない。

## Initial universe candidate

productionで固定する前の暫定ルール:

1. historical `equities_master` を当日dateで結合
2. Prime / Standard / Growth
3. ordinary-equity相当の `ProdCat` を公式仕様確認後に固定
4. OHLCとvolumeが当日利用可能
5. 最低売買代金条件は研究で複数thresholdを事前定義
6. delisted/relisted/security-code changesをcurrent masterで遡及補完しない

equal-weight全銘柄結果だけでなく、最低限:

- all eligible
- liquid universe
- Prime only
- market-cap weighted diagnostic

を別に出す。

## Quality summary

### 確認済み

- private packはsource manifestとSHA-256を持つ。
- normalized DuckDBの期待SHA-256が固定されている。
- base raw periodは2016-08-01〜2026-08-25。
- incremental update filesがbase cutoff後にも保存されている。
- bars/master/financial/TOPIX/calendarが利用可能。
- financial summaryには公表時刻がある。
- masterは日付付きであり、historical-universe構築に利用できる。

### 注意

- barsには商品種別を問わず多数のコードが含まれるため、master filter必須。
- 当日取引なし等でOHLC/volume欠損が存在する。
- `MktCap` は全行で埋まらない。
- `ExRT` は検査した日次snapshotでは全欠損。用途を推測しない。
- 配当込みtotal returnをこのpackだけで保証できるとは扱わない。
- 恒久的issuer/security identityを現在のcodeだけで保証しない。
- `AdjFactor` の適用方向・corporate-action処理は公式定義と実例でunit testする。
- financial summaryはevent row型であり、欠損を0へ変換しない。

## P0 acceptance checks

モデル作成前に以下を自動監査する。

- `(Date, Code)` duplicate
- OHLC price constraints
- negative/zero volume/value anomaly
- adjustment-factor jumps
- code-format changes
- historical master join rate
- market-section history
- next-trading-day target alignment
- delisting/relisting behavior
- missing OHLC reason
- extreme return around corporate actions
- 2024-11-05 TSE closing-auction regime split
- data cutoff / feature availability cutoff

## 未解決事項

1. `UL`, `LL`, `ExRT`, `ProdCat`, `Mrgn` の公式field definitionをversion固定して記録する。
2. `AdjFactor` のexact adjustment conventionをsplit事例で確認する。
3. normalized DuckDBの全table/schema/row countを復元後に監査する。
4. 2016-08開始による最初のlookback burn-inを定義する。
5. daily P0では財務を最初から投入せず、価格・出来高baseline後のincremental blockとして扱う。
6. 2026-08-25 base packとincremental updateの統合手順を固定する。
7. tick/order-book private packはP0とは分離し、daily modelがOOSを通過した後に検討する。

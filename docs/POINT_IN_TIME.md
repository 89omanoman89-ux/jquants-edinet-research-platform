# POINT_IN_TIME

## 原則

決算期末と「市場参加者が情報を利用できた日時」は別物として扱う。

研究時点 `observation_time` で使用可能な情報は原則として:

```text
available_at <= observation_time
```

を満たすものだけとする。

決算期末だけを使って情報を過去日に遡及適用しない。

## 必要な時刻・期間情報

必要に応じて以下を保持する。

- period_start
- period_end
- filed_at
- available_at
- superseded_at
- document_id
- revision_id
- version

`available_at` の定義はsourceごとに明文化する。

## 訂正報告

例:

```text
2025-06-20  元開示  R&D = 100
2025-07-10  訂正    R&D = 120
```

Point-in-Timeでは:

- 2025-06-20〜2025-07-09: 100
- 2025-07-10以降: 120

と扱う。

訂正によって過去に公表されていた値を削除・上書きしない。

## 時系列統合

J-QuantsとEDINETを単純な `join(code, fiscal_year)` だけで結合しない。

観測日時に対して、その時点までに利用可能だった最新情報を割り当てる。

必要に応じてASOF/temporal joinを使用する。

EDINET年次情報を日次株価テーブルへ物理的に毎日複製する必要はない。履歴テーブルを保持し、研究用データ生成時に結合する方式を優先する。

## Look-ahead validation

最低限以下をテストする。

- `available_at > observation_time` のレコードがGoldへ入らない
- 訂正版が訂正日前へ遡及しない
- 年次/半期/四半期のperiodと公表日時を混同しない
- feature計算でも未来時点データを参照しない

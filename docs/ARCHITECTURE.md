# ARCHITECTURE

## 基本構成

```text
J-Quants source ──┐
                  ├── Bronze ── Silver ── Gold
EDINET source ────┘
```

## Bronze

元データまたは可能な限り元データに近い状態を保存する。

目的:

- 原本保存
- provenance保持
- parser再実行
- 正規化ルール変更への対応
- 再現性確保

保持候補:

- source
- acquisition_timestamp
- API/version情報
- file name / document ID
- hash
- taxonomy version
- raw identifiers
- raw values

Bronzeを研究用canonical datasetとして直接利用しない。

## Silver

意味論を正規化した層。

主なentity/table候補:

- issuer
- security
- identifier_mapping
- filing
- financial_fact
- context
- dimension
- segment
- shareholder
- ownership_relationship

EDINETではraw conceptとcanonical conceptの両方を保持し、元データへ遡れるようにする。

## Gold

バックテスト、スクリーニング、ML、ファクター研究等で直接利用するPoint-in-Timeデータと派生特徴量。

GoldはSilverの履歴を壊さず、研究時点で利用可能だった情報だけを選択して生成する。

## 技術方針

既存設計に合理的な別方針がない限り:

- Python
- Polars: dataframe処理
- DuckDB: 分析SQL、ASOF/検証
- Parquet: 分析用保存形式

初期段階では、必要性が明確でないDBサーバー、クラウド基盤、コンテナ、分散処理基盤を導入しない。

## データ配置の例

```text
data/
├── jquants/    # 既存元データ、read-only
├── edinet/     # 取得したEDINET元データ
├── bronze/
├── silver/
└── gold/
```

実データはGitへコミットしない。

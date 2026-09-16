# DATA_MODEL

## Entity identity

会社名や証券コードだけを永久的な内部IDとして使用しない。

最低限、以下の内部IDを持つ。

- `issuer_id`: 発行体・企業を表す内部ID
- `security_id`: 上場証券を表す内部ID

## Identifier mapping

可能な範囲で以下を対応付ける。

- J-Quants Code
- EDINET code
- EDINET secCode
- 法人番号
- ISIN等の識別子
- raw company name
- normalized company name

identifier mappingには必要に応じて:

- `valid_from`
- `valid_to`
- `mapping_method`
- `mapping_confidence`
- `source`

を持たせる。

考慮するイベント:

- 証券コード変更
- 社名変更
- 上場廃止 / 再上場
- 合併
- 持株会社化
- 株式移転
- 組織再編

権威あるidentifierが利用できる場合、会社名のfuzzy matchingを第一選択にしない。

## Filing

候補schema:

```text
filing_id
doc_id
issuer_id
edinet_code
security_id
document_type
taxonomy_version
period_start
period_end
filed_at
available_at
superseded_at
revision_id
version
source_file
acquisition_timestamp
```

## Fact

横長テーブルだけを中核にせず、raw factを保持できる構造にする。

候補schema:

```text
fact_id
filing_id
issuer_id
security_id
raw_concept
canonical_concept
raw_value
raw_unit
normalized_value
normalized_unit
context_id
dimensions
period_start
period_end
consolidated_flag
source
normalization_status
```

## 欠損と0

欠損と0を明確に区別する。

可能な範囲で以下を区別する。

- explicit_zero
- not_disclosed
- not_applicable
- extraction_failed
- concept_unknown
- document_unavailable
- normalization_unresolved

sourceが明示的に0を表していない限り、欠損を0に変換しない。

## 単位

暗黙の単位変換を避ける。

canonical unitへ変換する場合も以下を追跡可能にする。

- raw_value
- raw_unit
- normalized_value
- normalized_unit
- conversion_rule

JPY、千円、百万円、株数、人数、割合等を混同しない。

## Segment

最低限:

- `raw_segment_name`
- `normalized_segment_id`

を分離する。

名称変更、統合、分割、組織再編に注意する。confidenceが低い正規化ではraw名称を優先する。

## Ownership

株主名はraw値を必ず保持する。

政策保有等では以下を混同しない。

- 当期 / 前期
- 特定投資株式
- みなし保有
- 純投資目的
- 政策保有目的
- 保有銘柄
- 株式数
- 評価額
- 保有目的

大量保有報告では提出者と投資対象会社を別entityとして扱う。

例:

```text
holder_id -> target_security_id -> ownership_pct
```

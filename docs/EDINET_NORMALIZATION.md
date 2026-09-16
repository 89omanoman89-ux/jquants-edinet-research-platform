# EDINET_NORMALIZATION

## 基本原則

concept名が似ているだけで同じ意味として扱わない。

canonical conceptへ変換する前に、少なくとも以下を確認する。

- 会計基準
- period
- context
- unit
- consolidated / non-consolidated
- dimensions
- taxonomy version
- source definition

JGAAPとIFRS等の違いを無視しない。

企業独自extension conceptを安易に既存conceptへ割り当てない。曖昧な場合はrawのまま保持する。

## Rawで保持する情報

可能な限り以下を保持する。

- doc_id
- document_type
- EDINET code
- security code
- submit datetime
- period start/end
- taxonomy version
- raw concept
- context id
- dimensions
- raw unit
- raw value
- consolidated / non-consolidated情報
- source file
- acquisition timestamp

未知conceptを黙って捨てず `unmapped` としてcoverage reportへ出す。

## Canonicalization status

正規化結果には状態を持たせることを検討する。

例:

- mapped_exact
- mapped_rule
- mapped_manual
- ambiguous
- unmapped
- rejected

## セグメント

raw segment nameを必ず保持する。

同一企業でも年度間で名称・構成が変化するため、自動的に永続的な同一事業と扱わない。

## 株主・政策保有

raw shareholder nameを保持する。

normalized nameを作成する場合は、変換根拠とconfidenceを追跡できるようにする。

政策保有は多次元データとして扱い、当期/前期、保有種別、銘柄、株数、評価額、保有目的を混同しない。

## 大量保有報告

提出者と対象企業を混同しない。

holder、target issuer/security、共同保有者、保有割合、保有目的、訂正・変更履歴を関係データとして扱う。

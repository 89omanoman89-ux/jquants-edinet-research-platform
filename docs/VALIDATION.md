# VALIDATION

Validationはパイプラインの必須構成要素とする。

## Identifier

- Code mapping率
- unmapped securities
- duplicate mapping
- validity period
- fuzzy mapping利用件数とconfidence

## Duplicate

想定primary keyで重複が存在しないか確認する。

重複が存在する場合、訂正、dimension違い、source違い等の意味を確認し、単純dropしない。

## Missingness

列、年度、企業、会計基準等でmissing率を確認する。

missingとexplicit zeroを区別する。

## Unit

- 想定外unit
- JPY / 千円 / 百万円
- 人数 / 株数 / 比率
- normalization前後の単位整合性

## Sign

費用、負債、CF等の符号が合理的か確認する。

異常値を自動削除せずreportする。

## Period

- 当期 / 前期
- instant / duration
- fiscal period
- period start/end

の混同を検出する。

## Scope

連結 / 個別を混同していないか確認する。

## Dimensions

segment等のdimensionを落としていないか確認する。

## Revision

訂正書類をversion管理し、過去値を上書きしていないか確認する。

## Source agreement

J-QuantsとEDINETに比較可能な値が存在する場合、差異を検証する。

一致すればparser validationに利用できる。不一致の場合は以下を確認する。

- definition
- source
- period
- consolidated scope
- accounting standard
- unit
- publication timing
- revision
- rounding
- fiscal period

原因不明のままcanonical valueへ強制統合しない。

## Point-in-Time

未来情報混入を検証する。

詳細は `POINT_IN_TIME.md` を参照する。

## Coverage Report

主要データセットについて可能な範囲で以下を出力する。

- 対象企業数
- 対象年度
- 対象書類数
- 正常抽出率
- missing率
- unmapped concept数
- identifier mapping率
- anomalous records
- validation failures

年度、会計基準、document type等でcoverageが大きく変わる場合はbreakdownを出す。

## テスト方針

変更内容に比例したテストを行う。

優先:

- unit test
- representative fixture
- regression test
- schema test
- Point-in-Time test
- parser test
- known-company validation

関連テストが通過した後、新しい変更、失敗、未解決問題がなければ同じ重いテストを無意味に繰り返さない。

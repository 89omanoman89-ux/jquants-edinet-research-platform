# AGENTS.md

## ミッション

既存のJ-Quantsデータを基盤として、J-Quantsにない、またはEDINETの方が研究上有用な粒度・詳細度を持つ情報を補完し、日本株の定量研究に利用できる高品質なPoint-in-Timeデータ基盤を構築する。

最優先事項は、正確性、再現性、出典追跡可能性、Point-in-Time整合性、検証可能性である。データ件数や特徴量数を増やすこと自体を目的にしない。

## 絶対ルール

- 既存J-Quants元データはread-onlyとして扱い、変更・上書き・削除・移動・リネームしない。
- 誤った値を埋めるより、欠損として残すことを優先する。
- 不明なXBRL concept、context、dimension、unit、issuer extensionを推測で正規化しない。
- raw dataとcanonical dataを分離し、元データへのtraceabilityを失わない。
- J-QuantsとEDINETの値が異なる場合、片方で黙って上書きせず差異を検証する。
- 決算期末と情報利用可能日時を区別する。
- 訂正報告書等で過去に公表されていた値を上書きしない。
- 研究用データは原則として `available_at <= observation_time` を満たす情報だけを利用する。
- 会社名だけを恒久的なjoin keyとして使わない。
- 不要なインフラ、全面リファクタリング、UI、Webサービス化、クラウド移行など、依頼範囲外の作業を勝手に追加しない。

## 最初に行うこと

EDINETの本格実装前に、既存J-Quantsデータを調査する。

調査結果は `docs/JQUANTS_INVENTORY.md` に記録する。

少なくとも以下を確認する。

- ファイル一覧と形式
- schemaとデータ型
- 行数、日付範囲、銘柄数
- primary key候補
- 欠損、重複、コード形式
- 対応するJ-Quantsデータセット/API
- 不明な列と未解決事項

その後、`docs/EDINET_COVERAGE.md` にJ-QuantsとEDINETのCoverage Matrixを作る。

分類は以下とする。

- A: J-Quantsで十分
- B: J-QuantsにもあるがEDINETに追加価値がある
- C: J-QuantsになくEDINETから取得価値が高い
- D: 意味解釈または正規化が不安定で要検証

Coverage Matrix完成前にEDINETを大量実装しない。

## データ層

原則としてBronze / Silver / Goldの3層に分離する。

- Bronze: 元データまたは最小限の加工データ。provenanceを保持する。
- Silver: issuer、security、filing、fact、segment、ownership等を正規化する。
- Gold: バックテスト、スクリーニング、ML、ファクター研究用のPoint-in-Timeデータと特徴量。

詳細は `docs/ARCHITECTURE.md` を参照する。

## Identifier

内部識別子として `issuer_id` と `security_id` を持つ。

J-Quants Code、EDINET code、EDINET secCode、法人番号等を可能な範囲で対応付け、必要に応じて `valid_from` / `valid_to` を保持する。

コード変更、社名変更、上場廃止、再上場、合併、持株会社化、株式移転等を考慮する。

詳細は `docs/DATA_MODEL.md` を参照する。

## EDINET

未知conceptを黙って捨てない。正規化できないものはrawのまま保持し、coverage reportで確認可能にする。

最初の補完候補は以下とする。

- 従業員数
- 平均給与
- 平均年齢
- 平均勤続年数
- 研究開発費
- 設備投資額
- 減価償却費

これらのcoverageとvalidationが十分になってから詳細財務、大株主、セグメント、政策保有株、大量保有報告、開示テキストへ拡張する。

詳細は `docs/EDINET_COVERAGE.md` と `docs/EDINET_NORMALIZATION.md` を参照する。

## Point-in-Time

最低限、必要に応じて以下を保持する。

- `period_start`
- `period_end`
- `filed_at`
- `available_at`
- `superseded_at`
- `document_id`
- `revision_id`
- `version`

単純な `join(code, fiscal_year)` だけで時系列統合しない。必要に応じてASOF/temporal joinを使用する。

詳細は `docs/POINT_IN_TIME.md` を参照する。

## Validation

Validationは後処理ではなくパイプラインの一部とする。

最低限以下を検証する。

- identifier mapping
- duplicate
- missingness
- unit
- sign
- period
- consolidated / non-consolidated
- dimensions
- revisions
- abnormal values
- source agreement
- Point-in-Time leakage

パイプラインが正常終了しただけでデータが正しいと判断しない。coverage reportとvalidation reportを作成する。

詳細は `docs/VALIDATION.md` を参照する。

## 作業方法

- リポジトリ、既存データ、公式仕様、既存ドキュメントから合理的に判断できる安全で可逆なローカル作業は自律的に進める。
- データ意味論を左右する曖昧さを根拠なく推測しない。
- 大きな機能追加やschema変更では `docs/PLANS.md` に従ってExecPlanを作成・更新する。
- 関連テストが通った後、新しい変更や失敗がない限り同じテストを無意味に繰り返さない。
- subagentは独立性が高く、並列化が品質または速度を改善する場合のみ使用する。
- 依頼範囲外の改善案は必要に応じてfuture workとして記録し、自動実装しない。

## 参照文書

重要な判断の前に関連文書を読むこと。

- `docs/GOALS.md`
- `docs/ARCHITECTURE.md`
- `docs/DATA_MODEL.md`
- `docs/JQUANTS_INVENTORY.md`
- `docs/EDINET_COVERAGE.md`
- `docs/EDINET_NORMALIZATION.md`
- `docs/POINT_IN_TIME.md`
- `docs/VALIDATION.md`
- `docs/FEATURES.md`
- `docs/PLANS.md`

詳細仕様をこのファイルへ大量に重複記載しない。このファイルは恒久ルールと参照先を示す案内図として扱う。

## 完了条件

コードを書いただけでは完了としない。少なくとも以下を満たすこと。

- 必要な実装が存在する
- 関連テストが成功している
- validationが実行されている
- coverageが確認されている
- provenanceとtraceabilityが保持されている
- Point-in-Time要件を満たしている
- 重要なschema判断が文書化されている
- 既知の曖昧点と未解決事項が記録されている
- 既存J-Quants元データが変更されていない

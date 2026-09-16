# ミッション

既存のJ-Quantsデータセットと、EDINETの補完的な開示情報を統合し、日本株の定量研究に利用できる高品質なPoint-in-Timeデータセットを構築する。

既存のJ-Quantsデータは重要な入力資産として扱う。

既存J-Quantsデータの元ファイルを、変更・上書き・削除・移動・リネームしてはならない。

EDINETを利用する目的は、J-Quantsですでに十分提供されているデータを再構築することではない。

J-Quantsには存在しない情報、またはJ-QuantsよりEDINETの方が研究上有用な粒度・詳細度を持つ情報を補完することを目的とする。

データ件数や特徴量数を増やすことよりも、以下を優先する。

- 正確性
- データ定義の明確性
- 出典追跡可能性
- Point-in-Time整合性
- 再現性
- 検証可能性

誤った値を埋めるより、欠損として残すことを優先する。

---

# 作業方針

リポジトリ、既存データ、既存ドキュメント、公的仕様書から合理的に判断できる内容については、自律的に調査・判断・実装を進める。

安全に実行可能なローカル作業について、単に提案だけして作業を止めてはならない。

与えられたタスクについて、調査・実装・検証・文書化まで、レビュー可能な状態まで進める。

ただし、以下の場合はユーザー確認を優先する。

- 結果を大きく左右する情報が不足している
- 公的資料や既存データから判断できない
- 複数の意味解釈があり、どれを選ぶかでデータ定義が変わる
- 元データの変更・削除等、不可逆な操作が必要
- 外部公開、デプロイ、課金を伴う操作が必要
- 利用規約やライセンス上の判断が必要

単なる実装上の細部については、合理的なデフォルトを採用し、必要に応じて判断内容を文書化して進める。

---

# 正式な情報源

アーキテクチャやデータ意味論について重要な判断を行う前に、関連する `docs/` 以下の文書を読むこと。

以下を基本的な正式文書とする。

- `docs/GOALS.md`
  - プロジェクトの目的
  - 対象範囲
  - 非対象範囲
- `docs/ARCHITECTURE.md`
  - システム構成
  - データフロー
  - ディレクトリ構造
- `docs/DATA_MODEL.md`
  - canonical schema
  - entity
  - fact
  - identifier
  - relationship
- `docs/JQUANTS_INVENTORY.md`
  - 既存J-Quantsデータの実際の収録内容
  - ファイル一覧
  - 列定義
  - 期間
  - coverage
- `docs/EDINET_COVERAGE.md`
  - EDINETから補完する項目
  - 補完しない項目
  - 抽出難易度
  - データ定義
- `docs/POINT_IN_TIME.md`
  - 情報利用可能時点
  - 訂正書類
  - ASOF処理
  - バックテスト時の時間制約
- `docs/VALIDATION.md`
  - 品質検証ルール
  - coverage検証
  - 異常値検証
- `docs/DATA_DICTIONARY.md`
  - 各項目の正式定義
  - source
  - concept
  - unit
  - calculation
- `docs/PLANS.md`
  - 大規模タスク用ExecPlanの作成・更新方法

詳細情報をこの `AGENTS.md` に大量に重複記載しない。

このファイルは、プロジェクト全体の原則と参照先を示す「案内図」として扱う。

既存データと文書の内容が矛盾する場合、どちらかを黙って採用してはならない。

差異を調査し、原因と判断を記録する。

---

# 既存J-Quantsデータの扱い

既存のJ-Quants元データはread-onlyとして扱う。

以下を禁止する。

- 上書き
- 削除
- リネーム
- 元ファイルへの加工結果の保存
- 元ファイル内の値変更

EDINET実装を本格的に開始する前に、既存J-Quantsデータの実態を調査すること。

少なくとも以下を調査する。

- ファイル名
- ファイル形式
- 行数
- 列名
- データ型
- 最小日付
- 最大日付
- 銘柄数
- primary key候補
- 欠損率
- 重複
- コード形式
- 更新頻度
- データの意味
- 対応するJ-Quantsデータセット/API

J-Quantsに項目が存在しないと判断する前に、実際の既存データを確認すること。

列名だけを見て意味を断定しない。

不明な列については、可能であればJ-Quants公式仕様と照合する。

J-Quantsですでに十分提供されているデータについて、EDINETから同一データを本番用として再構築しない。

ただし、以下の場合はEDINET側も保持してよい。

- データ品質検証
- definition差分検証
- 粒度差の比較
- J-Quantsより詳細な内訳が存在する
- 将来の研究に必要な出典保存

---

# J-QuantsとEDINETのCoverage分類

EDINETからデータを実装する前に、候補項目を以下の4区分に分類する。

## A: J-Quantsで十分取得可能

J-Quantsの既存データで研究用途として十分な情報。

原則としてEDINETから本番用再構築を行わない。

## B: J-Quantsにも存在するがEDINETに追加価値がある

例:

- より詳細なBS/PL/CF
- 内訳
- セグメント
- 詳細な注記

両方のsourceを保持する。

## C: J-Quantsには存在しない、または著しく不足している

EDINETから構造化取得する価値が高い情報。

優先的に実装する。

## D: 意味解釈や正規化が不安定

即座にcanonical fieldへ変換しない。

raw dataを保存し、意味論を検証してから正規化する。

曖昧なconceptを推測で既存項目へ割り当ててはならない。

---

# EDINETで優先的に検討する情報

少なくとも以下についてCoverage Matrixを作成する。

## 人的資本

- 従業員数
- 臨時雇用者数
- 平均給与
- 平均年齢
- 平均勤続年数

## 投資・研究開発

- 研究開発費
- 設備投資額
- 減価償却費

## 詳細財務

- 現金及び預金
- 棚卸資産
- 売掛金
- 投資有価証券
- 短期借入金
- 長期借入金
- 社債
- のれん
- 退職給付関連項目
- 詳細CF項目

## セグメント

- セグメント売上
- セグメント利益
- セグメント資産
- セグメント従業員
- セグメント研究開発費
- セグメント設備投資

## 所有構造

- 大株主
- 上位株主保有比率
- 政策保有株式
- みなし保有
- 保有株式数
- 貸借対照表計上額

## ガバナンス

- 役員
- ガバナンス関連情報

## 開示テキスト

- 事業等のリスク
- MD&A
- 経営方針
- 事業内容
- サステナビリティ

## 大量保有報告

- 保有者
- 対象企業
- 保有割合
- 共同保有者
- 保有目的
- 変更履歴

すべてを最初から本番実装してはならない。

取得難易度、意味の安定性、研究価値を評価した上で段階的に実装する。

---

# Entity / Identifier設計

会社名を永久的なjoin keyとして使用してはならない。

J-Quantsの証券コードだけを永久IDとして使用してはならない。

EDINETコードだけを永久IDとして使用してはならない。

内部用のidentifierを設計する。

最低限、

- `issuer_id`
- `security_id`

を持つ。

可能な範囲で以下を対応付ける。

- J-Quants Code
- EDINET code
- EDINET secCode
- 法人番号
- ISIN等の識別子
- 会社名
- 銘柄名

identifier mappingには必要に応じて、

- `valid_from`
- `valid_to`

を持たせる。

以下のイベントに対応可能な設計とする。

- 証券コード変更
- 社名変更
- 上場廃止
- 再上場
- 合併
- 持株会社化
- 株式移転
- 組織再編

権威あるidentifierが利用できる場合、会社名のfuzzy matchingを第一選択にしてはならない。

fuzzy matchingを使用した場合は、その事実とconfidenceを記録する。

---

# Bronze / Silver / Gold

データは原則として3層に分離する。

## Bronze

取得した元データ、または可能な限り元データに近い状態。

目的:

- 原本保存
- 再現性
- parser再実行
- 後からの正規化ルール変更

可能な限り以下を保持する。

- source
- acquisition timestamp
- API version
- file name
- document ID
- hash
- taxonomy version
- raw value
- raw identifier

Bronzeを分析用canonical datasetとして直接利用しない。

## Silver

意味を正規化したデータ。

例:

- issuer
- security
- filing
- fact
- context
- dimension
- segment
- shareholder
- ownership relationship

EDINETについては、

- raw concept
- canonical concept

の両方を可能な限り保持する。

元データへのtraceabilityを失ってはならない。

## Gold

バックテスト、スクリーニング、機械学習、ファクター研究等で直接利用できるデータ。

Goldは原則Point-in-Timeで構築する。

特徴量のsourceと計算方法を追跡可能にする。

---

# EDINETのRawデータ

EDINETのXBRL等を解析する場合、可能な限り以下を保存する。

- `doc_id`
- document type
- EDINET code
- security code
- submit datetime
- period start
- period end
- taxonomy version
- raw concept
- context id
- dimensions
- unit
- raw value
- consolidated / non-consolidated情報
- source file
- acquisition timestamp

未知のXBRL conceptを黙って削除しない。

正規化できないconceptは、

`unmapped concept`

として保存する。

coverage reportで確認可能にする。

issuer extension taxonomyを標準taxonomyと区別する。

---

# Canonicalization

concept名が似ているという理由だけで同じ意味として扱ってはならない。

以下を確認した上でcanonical conceptへ変換する。

- 会計基準
- period
- context
- unit
- consolidated / non-consolidated
- dimensions
- taxonomy
- source definition

JGAAPとIFRS等の違いを無視しない。

企業独自extension conceptを安易に既存conceptへマッピングしない。

曖昧な場合はraw conceptを残す。

---

# Point-in-Time原則

このプロジェクトではPoint-in-Time整合性を最重要制約の1つとする。

決算期末と「市場参加者がその情報を利用できた日時」は別物として扱う。

必要に応じて以下を保持する。

- `period_start`
- `period_end`
- `filed_at`
- `available_at`
- `superseded_at`
- `document_id`
- `revision_id`
- `version`

研究時点 `observation_time` において利用してよい情報は原則として、

`available_at <= observation_time`

を満たす情報のみとする。

決算期末日だけを使ってデータを過去日に遡及適用してはならない。

---

# 訂正報告書

訂正報告書等によって値が変更された場合、過去の値を削除・上書きしてはならない。

例:

元開示:

- filed\_at = 2025-06-20
- R&D = 100

訂正:

- filed\_at = 2025-07-10
- R&D = 120

この場合、

2025-06-20から2025-07-09までのPoint-in-Time値は100。

2025-07-10以降は120。

過去に公表されていた値を後から書き換えない。

必要に応じて有効期間を管理する。

---

# 時系列統合

J-QuantsとEDINETを単純な、

`join(code, fiscal_year)`

だけで統合してはならない。

時間依存する情報については、必要に応じてASOF JOIN等を使用する。

原則:

各観測日に対して、その日時点までに利用可能だった最新情報を割り当てる。

日次株価テーブルへEDINETの年次情報を毎日物理的に複製する必要はない。

適切な時点テーブルを保持し、研究用データ生成時に結合する方式を優先する。

---

# J-QuantsとEDINETの値が異なる場合

片方を自動的に「正しい」と判断して上書きしてはならない。

以下を調査する。

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

比較可能な項目については差異をvalidation reportへ出力する。

例:

- J-Quants revenue
- EDINET revenue

一致すればparser検証に利用できる。

不一致の場合、原因を調査する。

原因が不明なままcanonical valueへ統合しない。

---

# 欠損値

欠損値と0を明確に区別する。

情報が存在しない場合、

0

として保存してはならない。

0はsourceが明示的に0を表している場合のみ使用する。

以下を可能な限り区別する。

- value = 0
- not disclosed
- not applicable
- extraction failed
- concept unknown
- document unavailable
- normalization unresolved

必要に応じてstatus列を設ける。

---

# 単位

単位変換を暗黙に行わない。

元unitを保存する。

canonical unitへ変換した場合、

- raw value
- raw unit
- normalized value
- normalized unit

を追跡可能にする。

JPY、千円、百万円等の混同を防ぐ。

株数、人数、割合、金額を明確に区別する。

---

# セグメント

セグメント名を自動的に永続的な同一事業として扱ってはならない。

最低限、

- raw segment name
- normalized segment identifier

を分離する。

名称変更、組織再編、セグメント統合・分割に注意する。

自動正規化のconfidenceが低い場合は、raw名称を優先して保持する。

---

# 大株主・政策保有株

株主名には表記揺れが存在するため、raw shareholder nameを必ず保持する。

normalized shareholder nameを作成する場合もraw値を削除しない。

政策保有株については、以下のdimensionsを混同しない。

- 当期 / 前期
- 特定投資株式
- みなし保有
- 純投資目的
- 政策保有目的
- 保有銘柄
- 株式数
- 評価額
- 保有目的

多次元データとして設計する。

---

# 大量保有報告書

大量保有報告書では、提出者と投資対象会社を混同しない。

以下を別entityとして扱う。

- holder
- target issuer/security

関係データとして保存する。

例:

`holder_id -> target_security_id -> ownership_pct`

訂正、変更報告、共同保有者をversion管理する。

EDINET APIの提出者情報だけを対象企業コードとして利用してはならない。

---

# データ保存

既存設計に明確な別方針がない限り、分析用データはParquetを基本とする。

Pythonを使用する。

dataframe処理は原則Polarsを優先する。

分析SQLにはDuckDBを優先する。

ただし、既存リポジトリに合理的な技術選定が存在する場合、無意味に書き換えない。

不要なデータベースサーバー、クラウド基盤、コンテナ、分散処理基盤等を導入しない。

現在のデータ規模と要件で必要性が証明されてから導入する。

---

# Validation

Validationは後処理ではなく、データパイプラインの必須構成要素とする。

最低限以下を検証する。

## Identifier

- Code mapping率
- unmapped securities
- duplicate mapping
- validity period

## 重複

想定primary keyで重複が存在しないか確認する。

## Missingness

列単位、年度単位、企業単位で確認する。

## Unit

想定外unitを検出する。

## Sign

費用・負債・CF等の符号が合理的か検証する。

## Period

当期 / 前期の混同を検出する。

## Scope

連結 / 個別を混同していないか検証する。

## Dimensions

segment等のdimensionを落としていないか確認する。

## Revision

訂正書類を適切にversion管理しているか確認する。

## Abnormal values

明らかに異常な桁数や値を検出する。

ただし、異常値を自動的に削除してはならない。

## Source agreement

J-QuantsとEDINETに比較可能な値が存在する場合、差異を検証する。

## Point-in-Time

未来情報が混入していないことを検証する。

---

# Coverage Report

「パイプラインがエラーなく動いた」ことを「データが正しい」と解釈してはならない。

各主要データセットについてcoverage reportを作成する。

可能な範囲で以下を含める。

- 対象企業数
- 対象年度
- 対象書類数
- 正常抽出率
- missing率
- unmapped concept数
- identifier mapping率
- anomalous records
- validation failures

年度や会計基準等でcoverageが大きく変わる場合、それも確認できるようにする。

---

# テスト方針

変更内容に比例したテストを行う。

小さな変更に対してプロジェクト全体の重いテストを毎回無条件に実行しない。

以下を優先する。

- unit test
- representative fixture
- regression test
- schema test
- Point-in-Time test
- parser test
- known-company validation

関連テストが正常に通過した後、新しいコード変更・失敗・未解決問題が存在しない限り、同じテストを無意味に繰り返さない。

実装コードと同じロジックを単純に複製しただけのテストを大量に作らない。

テストは、実際のデータ意味論や境界条件を検証するものとする。

---

# Subagentの利用

Subagentは、独立性が高く、並列化することで速度または品質が向上する場合に使用する。

適した例:

- J-Quants既存データの棚卸し
- EDINET公式仕様の調査
- XBRL taxonomy調査
- 独立したvalidation review
- データモデルのレビュー

適さない例:

- 単純なファイル編集
- 連続して実行すべき小作業
- 親agentだけで十分処理可能な作業

不必要に多数のsubagentを生成しない。

recursive delegationを安易に行わない。

親タスク完了前に、必要なsubagent結果を回収・検証する。

不要になったsubagentの作業を放置しない。

---

# スコープ管理

「改善できそうだから」という理由だけでプロジェクト範囲を拡張しない。

現在の目的に直接必要ない改善案は、必要に応じてfuture workとして記録する。

ユーザーから要求されていない以下を勝手に行わない。

- 全面リファクタリング
- インフラ刷新
- データベース移行
- unrelated bug修正
- UI開発
- Webサービス化
- クラウド移行

問題がある場合は、依頼範囲内で根本原因を修正する。

---

# ExecPlan

大規模な機能追加、schema変更、複数段階のデータ取得・正規化処理を実装する場合、`docs/PLANS.md` に従ってExecPlanを作成・更新する。

ExecPlanには最低限以下を記載する。

- 目的
- 調査した資料
- 現状
- 仮定
- 対象範囲
- 非対象範囲
- schema上の判断
- 実装段階
- acceptance criteria
- validation方法
- validation結果
- 未解決事項

計画文書は一度作って終わりではなく、実装状況に応じて更新する。

安全かつ可逆なローカル作業について、各Phaseごとにユーザー承認を要求して作業を止めない。

ただし、重要な意味論が確定できない場合は推測で進めない。

---

# 初期実装の優先順序

最初からEDINET全項目を実装しない。

まず既存J-Quantsの棚卸しを完了する。

その後、最初のEDINET補完として以下を優先する。

1. 従業員数
2. 平均給与
3. 平均年齢
4. 平均勤続年数
5. 研究開発費
6. 設備投資額
7. 減価償却費

これらについて十分なcoverageとvalidationが確認できた後、次の段階へ進む。

次候補:

- 詳細BS/PL/CF
- 大株主

その後:

- セグメント
- 政策保有株式

最後に:

- 大量保有報告
- 開示テキスト
- NLP / embedding等

各段階で品質確認を行う。

---

# 特徴量生成

Gold層では、取得したraw値そのものだけでなく、研究用特徴量を生成できる設計にする。

例:

## Human Capital

- revenue\_per\_employee
- operating\_profit\_per\_employee
- employee\_growth
- salary\_growth
- revenue\_growth\_minus\_employee\_growth

## Investment

- rnd\_to\_sales
- rnd\_growth
- capex\_to\_sales
- capex\_to\_assets
- capex\_to\_depreciation
- investment\_acceleration

## Balance Sheet

- net\_debt\_to\_equity
- goodwill\_to\_equity
- investment\_securities\_to\_equity
- inventory\_growth\_minus\_sales\_growth

## Ownership

- top1\_ownership
- top5\_ownership
- top10\_ownership
- ownership\_concentration
- cross\_shareholding\_to\_equity
- cross\_shareholding\_reduction

## Segment

- segment\_hhi
- largest\_segment\_share
- fastest\_segment\_growth
- segment\_profit\_dispersion
- segment\_growth\_dispersion

## Text

将来的に、

- risk\_text\_change
- MD&A similarity
- new risk topics
- removed risk topics

等を追加できる設計とする。

特徴量は必ず元データと計算定義を追跡可能にする。

---

# 重要原則

このプロジェクトでは以下を常に守る。

1. 誤った値より欠損値を選ぶ。
2. 意味が分からない値を推測で正規化しない。
3. 元データへのtraceabilityを失わない。
4. 未来情報を混入させない。
5. 既存J-Quants元データを破壊しない。
6. source間の差異を黙って消さない。
7. raw dataとcanonical dataを分離する。
8. schemaとデータ定義を文書化する。
9. パイプライン成功だけで品質を判断しない。
10. 必要以上にプロジェクト範囲を拡大しない。

---

# 完了条件

コードを書いただけではタスク完了とみなさない。

以下を満たした場合に完了とする。

- 必要な実装が存在する
- 関連テストが成功している
- validationが実行されている
- coverageが確認されている
- provenanceが保存されている
- Point-in-Time要件を満たしている
- 元データへのtraceabilityがある
- 重要なschema判断が文書化されている
- 既知の曖昧点や未解決事項が記録されている
- 既存J-Quants元データが変更されていない

データ量の多さではなく、研究に安心して使用できることを完成基準とする。

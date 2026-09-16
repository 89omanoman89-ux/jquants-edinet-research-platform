# PLANS

大規模な機能追加、schema変更、複数段階のデータ取得・正規化ではExecPlanを作成・更新する。

## ExecPlanに含める項目

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

計画は実装中も更新するliving documentとする。

## 初期ロードマップ

### Phase 1: J-Quants棚卸し

- 元データをread-onlyで調査
- schema / coverage / qualityを文書化
- identifier候補を確認

成果物:

- `JQUANTS_INVENTORY.md`

### Phase 2: EDINET Coverage設計

- J-QuantsとEDINETのCoverage Matrix作成
- A/B/C/D分類
- 初期実装対象の確定

成果物:

- `EDINET_COVERAGE.md`

### Phase 3: 基盤schema

- issuer/security identifier
- filing
- raw fact
- canonical fact
- Point-in-Time

成果物:

- `DATA_MODEL.md`
- `POINT_IN_TIME.md`

### Phase 4: 初期EDINET補完

候補:

- 従業員数
- 平均給与
- 平均年齢
- 平均勤続年数
- 研究開発費
- 設備投資額
- 減価償却費

### Phase 5: Validation

- J-Quants共通項目でparser validation
- coverage report
- missingness / unit / scope / revision検証
- Point-in-Time test

### Phase 6: 拡張

品質確認後、必要性に応じて:

- 詳細BS/PL/CF
- 大株主
- セグメント
- 政策保有株
- 大量保有報告
- 開示テキスト
- NLP/embedding

へ拡張する。

## Scope control

「改善できそうだから」という理由だけでスコープを広げない。

依頼範囲外の改善はfuture workとして記録し、必要性が確認されてから実装する。

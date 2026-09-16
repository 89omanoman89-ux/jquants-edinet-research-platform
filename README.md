# J-Quants × EDINET 日本株リサーチ基盤

既存のJ-Quantsデータセットに、EDINETから取得した補完情報を統合し、日本株の定量研究、バックテスト、機械学習、ファクター研究に利用できるPoint-in-Timeデータ基盤を構築するためのリポジトリです。

## 方針

- J-Quantsを市場データと既存構造化データの基盤として利用する。
- EDINETは、J-Quantsにない、またはより詳細な情報の補完に利用する。
- 元データ、正規化データ、研究用データをBronze / Silver / Goldに分離する。
- 未来情報混入を防ぐため、開示日時と訂正履歴を保持する。
- 誤った値を埋めるより欠損を優先する。
- provenanceと再現性を重視する。

## リポジトリ構成

```text
.
├── AGENTS.md
├── README.md
├── .gitignore
├── data/
│   └── README.md
├── docs/
│   ├── GOALS.md
│   ├── ARCHITECTURE.md
│   ├── DATA_MODEL.md
│   ├── JQUANTS_INVENTORY.md
│   ├── EDINET_COVERAGE.md
│   ├── EDINET_NORMALIZATION.md
│   ├── POINT_IN_TIME.md
│   ├── VALIDATION.md
│   ├── FEATURES.md
│   └── PLANS.md
└── archive/
    └── ORIGINAL_SPEC.md
```

## 開発開始時の順序

1. `AGENTS.md` を読む。
2. 既存J-Quantsデータをread-onlyで棚卸しする。
3. `docs/JQUANTS_INVENTORY.md` を更新する。
4. `docs/EDINET_COVERAGE.md` のCoverage Matrixを埋める。
5. `docs/DATA_MODEL.md` と `docs/POINT_IN_TIME.md` を実データに合わせて確定する。
6. 小さなEDINET補完項目から実装する。
7. coverageとvalidationを確認してから対象を拡張する。

## データについて

J-QuantsやEDINETから取得した実データ、大容量のCSV/Parquet、DuckDBファイル等は原則としてGitにコミットしません。`data/` はローカル作業領域として使用し、実データは `.gitignore` で除外します。

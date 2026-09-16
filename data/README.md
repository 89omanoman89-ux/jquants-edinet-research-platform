# data/

このディレクトリはローカルのデータ作業領域です。

J-QuantsやEDINETの実データ、大容量CSV/Parquet、DuckDBファイル等は原則としてGitHubへコミットしません。

想定構成:

```text
data/
├── jquants/   # 既存J-Quants元データ。read-only
├── edinet/    # EDINET取得元データ
├── bronze/
├── silver/
└── gold/
```

既存J-Quants元データは変更・削除・上書きしないでください。

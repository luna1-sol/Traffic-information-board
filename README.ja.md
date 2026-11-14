# Traffic Information Board

[English](README.md) | 日本語

バスや電車の次の発車時刻を大型モニターに表示するためのシステムです。

- 表示ボード（`/`）：次の発車予定を電光掲示板風に表示します。
- 管理画面（`/admin`）：CSV ファイルを取り込んで時刻表データを差し替えます。

## 技術スタック

| 構成要素 | 使用技術 |
| --- | --- |
| フロントエンド | Vite / React / TypeScript / Tailwind CSS |
| バックエンド | Go / chi / SQLite (modernc.org/sqlite) |

データの流れ：CSV → SQLite → Go API → React UI

## ディレクトリ構成

```text
digital-board/
├── backend/
│   ├── cmd/server/          # エントリーポイント（ルーター設定と起動）
│   ├── internal/db/         # SQLite 接続、スキーマ作成、初期データ投入
│   ├── internal/timetable/  # 型定義、DB クエリ、API ハンドラー、CSV 取り込み
│   ├── internal/webui/      # React ビルドの埋め込みと SPA フォールバック
│   └── data/timetable.db    # SQLite データベース（初回起動時に作成、Git 管理対象外）
├── frontend/
│   ├── src/pages/           # BoardPage（表示ボード）/ AdminPage（管理画面）
│   ├── src/components/      # 表示ボードの UI コンポーネント
│   ├── src/api/             # API クライアント
│   └── vite.config.ts       # React のビルド設定
├── scripts/
│   └── build-release.sh     # macOS / Ubuntu 向けクロスビルドスクリプト
├── dist/                    # リリース用バイナリの出力先
└── docs/
    ├── build.md             # macOS / Ubuntu でのビルド手順
    └── sample.csv           # 時刻表 CSV のサンプル
```

## 必要環境

- Go 1.26 以降
- Node.js 20 以降（npm を含む）

<a id="csv形式"></a>

## CSV 形式

時刻表は [docs/sample.csv](docs/sample.csv) を参考に作成してください。ファイルは UTF-8 の CSV で保存します。1 行目には次のヘッダーを、この順番のとおりに記述する必要があります。

```csv
kind,route_name,destination,departure_time,platform
```

2 行目以降に、1 行につき 1 本の発車予定を記述します。例：

```csv
kind,route_name,destination,departure_time,platform
bus,○○線,○○駅,07:40,○○前
train,○○本線,○○方面,07:27,○○駅
```

| 列 | 必須 | 説明 |
| --- | --- | --- |
| `kind` | 必須 | 交通手段の種別。`bus` または `train` のみ指定できます。 |
| `route_name` | 必須 | 表示ボードに表示する路線名・系統名。 |
| `destination` | 必須 | 表示ボードに表示する行き先。 |
| `departure_time` | 必須 | 発車時刻。`07:40` や `15:25` のように、ゼロ埋めした 24 時間表記の `HH:MM` 形式で指定します。 |
| `platform` | 必須 | バスの場合はバス停・乗り場、電車の場合は出発駅名。 |

注意事項：

- 列の追加・削除・名前の変更・並べ替えはしないでください。
- その日に必要なすべての発車予定を 1 つのファイルにまとめてください。
- CSV を取り込むと、既存の時刻表はすべて置き換えられます。
- UTF-8 で保存すれば日本語も使用できます。Excel で書き出す場合は「CSV UTF-8」形式を選択してください。
- `/admin` から取り込んだ後は、取り込まれた件数と表示される時刻表が正しいことを確認してください。

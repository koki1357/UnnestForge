# UnnestForge

```
 _   _                       _   _____                    
| | | |_ __  _ __   ___  ___| |_|  ___|__  _ __ __ _  ___ 
| | | | '_ \| '_ \ / _ \/ __| __| |_ / _ \| '__/ _` |/ _ \
| |_| | | | | | | |  __/\__ \ |_|  _| (_) | | | (_| |  __/
 \___/|_| |_|_| |_|\___||___/\__|_|  \___/|_|  \__, |\___|
                                               |___/     
    Forge Bulk Data with PostgreSQL Unnest Power
```

PostgreSQL用の高性能テストデータ生成ツールです。DDL解析からYAML設定生成、PostgreSQL `unnest`構文を使った超高速バルクインサートまで一貫してサポートします。

## 🚀 主な機能

### 1. DDL自動解析
- CREATE TABLE文から自動的にテーブル構造を解析
- PRIMARY KEY、UNIQUE制約、CHECK制約、NOT NULL制約を検出
- YAML設定ファイルを自動生成

### 2. 柔軟なデータ生成
- **テーブルレベル設定**: 日付・タイムスタンプのデフォルト値を一括設定
- **カラム個別設定**: 特定カラムの固定値指定
- **制約対応**: 文字数制限、CHECK制約、重複回避を自動処理
- **リアルなデータ**: 書籍、著者、レビューなど実用的なテストデータを生成

### 3. 高速バルク出力（PostgreSQL Unnest特化）
- **PostgreSQL unnest構文**: 10,000件/バッチの超高速インサート
- **CSV出力**: 各テーブル個別のCSVファイル生成
- **デバッグモード**: 詳細ログで問題解決をサポート

## 📁 ディレクトリ構造

```
unnest-forge/
├── src/main/java/com/unnestforge/
│   └── PostgreSQLBulkDataGenerator.java
├── input/          # DDLファイル配置
├── config/         # 生成されたYAML設定
├── output/         # 生成されたSQLファイル
├── datasets/       # 生成されたCSVファイル
└── build/          # コンパイル済みクラス
```

## 🛠️ セットアップ

```bash
# プロジェクトディレクトリ作成
mkdir unnest-forge && cd unnest-forge

# ソースコード配置
mkdir -p src/main/java/com/unnestforge

# コンパイル
mkdir -p build
javac -d build src/main/java/com/unnestforge/PostgreSQLBulkDataGenerator.java
```

## 🎯 使用方法

### ステップ1: DDL解析（設定ファイル生成）

```bash
# 1. DDLファイルをinput/ディレクトリに配置
cp your_schema.sql input/

# 2. DDL解析実行
java -cp build com.unnestforge.PostgreSQLBulkDataGenerator

# → config/your_schema.yaml が自動生成される
```

### ステップ2: 設定ファイルカスタマイズ

生成されたYAMLファイルを要件に合わせて編集：

```yaml
tables:
  books:
    records: 10000
    # テーブル全体の日付設定
    default_date: "2024-01-15"
    default_timestamp: "2024-01-15 09:00:00"
    
    columns:
      book_id:
        type: bigint
        start_from: 1000  # 既存データとの重複回避
        
      publication_date:
        type: date
        # → default_date ("2024-01-15") が使用される
        
      created_date:
        type: date
        # → default_date ("2024-01-15") が使用される
        
      special_edition_date:
        type: date
        fixed_value: "2020-01-01"  # このカラムだけ個別指定
        
      updated_at:
        type: timestamp(3)
        # → default_timestamp が使用される
```

### ステップ3: データ生成

```bash
# SQL生成（全テーブル一括）
java -cp build com.unnestforge.PostgreSQLBulkDataGenerator config/your_schema.yaml sql

# CSV生成（各テーブル個別）
java -cp build com.unnestforge.PostgreSQLBulkDataGenerator config/your_schema.yaml csv

# デバッグモード（詳細ログ出力）
java -cp build com.unnestforge.PostgreSQLBulkDataGenerator config/your_schema.yaml sql --debug
```

## ⚙️ 設定オプション

### テーブルレベル設定

| 設定項目 | 説明 | 例 |
|---------|------|---|
| `records` | 生成レコード数 | `10000` |
| `default_date` | 全dateカラムのデフォルト値 | `"2024-01-15"` |
| `default_timestamp` | 全timestampカラムのデフォルト値 | `"2024-01-15 09:00:00"` |

### カラムレベル設定

| 設定項目 | 説明 | 例 |
|---------|------|---|
| `type` | データ型（必須） | `varchar(100)`, `integer` |
| `start_from` | 開始値（PK/UK重複回避） | `1000` |
| `fixed_value` | 固定値（最優先） | `"2020-01-01"` |

### 設定の優先順位

1. **`fixed_value`** - カラム個別の固定値（最優先）
2. **`default_date`/`default_timestamp`** - テーブルレベルのデフォルト値
3. **システムデフォルト** - 自動生成ロジック（フォールバック）

## 📊 データ生成ルール

### 数値型
- **ID系**: `start_from + index` または連番
- **年カラム**: 2020-2030の循環
- **その他**: インデックスベースの連番

### 文字列型
- **varchar(1)**: "1", "2", "3"...
- **varchar(2)**: "01", "02", "03"...
- **varchar(4)**: "0001", "0002", "0003"...
- **varchar(5-10)**: "Book1", "Book2"... (プレフィックス+数字)
- **長い文字列**: "Book_1", "Author_2"... (最大長で切り詰め)
- **email形式**: "data_1@example.com", "data_2@example.com"...

### 日付・時刻型
- **テーブル設定あり**: `default_date`/`default_timestamp` を使用
- **システムデフォルト**: 有効な日付・時刻を自動生成

### その他
- **boolean**: true/false交互
- **decimal**: index * 100.0

## 🎮 実践例

### 出版業界向けテストデータ

```yaml
tables:
  books:
    records: 1000
    default_date: "2024-01-15"  # 出版日
    default_timestamp: "2024-01-15 09:00:00"
    
    columns:
      book_id:
        type: integer
        start_from: 1000  # 既存書籍IDとの重複回避
        
      publication_date:
        type: date
        # → 2024-01-15
        
      isbn:
        type: varchar(13)
        # → 13桁のISBN形式で生成
```

### 複数環境対応（本番データとの重複回避）

```yaml
tables:
  authors:
    records: 5000
    
    columns:
      author_id:
        type: bigint
        start_from: 100000  # 本番ID範囲(1-50000)との重複回避
        
      email:
        type: varchar(100)
        # → data_100001@example.com 形式で生成
        
      pen_name:
        type: varchar(50)
        # → "Author_100001" 形式で生成
```

## 🔧 トラブルシューティング

### よくある問題

#### 1. 文字数制限エラー
```
ERROR: value too long for type character varying(2)
```
**解決**: 自動的に制約に合わせてデータが生成されます

#### 2. PRIMARY KEY重複エラー
```
ERROR: duplicate key value violates unique constraint
```
**解決**: `start_from: 1000` を設定して既存データとの重複を回避

#### 3. CHECK制約違反
```
ERROR: new row violates check constraint
```
**解決**: 年カラム等は自動的に有効な値（2020-2030）が生成されます

#### 4. 日付範囲エラー
```
ERROR: date/time field value out of range: "2024-02-30"
```
**解決**: 自動的に有効な日付（1-28日）が生成されます

### デバッグモード

詳細なログでプロセスを確認：

```bash
java -cp build com.unnestforge.PostgreSQLBulkDataGenerator config/schema.yaml sql --debug
```

## 📝 UnnestForgeが生成するSQL例

UnnestForgeの最大の特徴は、PostgreSQLの`unnest`関数を活用した超高速バルクインサートです：

```sql
-- PostgreSQL unnest バルクインサート（10,000件/バッチ）
INSERT INTO books (book_id, title, publication_date, price, in_stock)
SELECT 
       unnest(ARRAY[1000, 1001, 1002, ...]::BIGINT[]),
       unnest(ARRAY['Book_1000', 'Book_1001', 'Book_1002', ...]::VARCHAR[]),
       unnest(ARRAY['2024-01-15', '2024-01-15', '2024-01-15', ...]::DATE[]),
       unnest(ARRAY[1500.0, 2000.0, 2500.0, ...]::DECIMAL[]),
       unnest(ARRAY[true, false, true, ...]::BOOLEAN[]);

-- 従来の個別INSERT文と比較して約10-50倍高速
```

### なぜUnnestが高速なのか？

1. **単一SQL文**: 10,000件を1つのINSERT文で処理
2. **配列効率**: PostgreSQLの内部配列処理を活用
3. **トランザクション最適化**: コミット回数を大幅削減
4. **メモリ効率**: バッチ処理でメモリ使用量を制御

## 🚀 高度な使用例

### マルチテーブル書籍管理データ生成

```yaml
tables:
  # 著者マスタ
  authors:
    records: 1000
    default_date: "2024-01-15"
    columns:
      author_id:
        type: integer
        start_from: 10000
      registration_date:
        type: date
        # → 2024-01-15
      name:
        type: varchar(100)
        # → "Author_10000" 形式
  
  # 書籍データ
  books:
    records: 10000
    default_date: "2024-01-15"
    default_timestamp: "2024-01-15 09:00:00"
    columns:
      book_id:
        type: bigint
        start_from: 100000
      author_id:
        type: integer
        start_from: 10000  # 著者マスタと整合
      publication_date:
        type: date
        # → 2024-01-15
      title:
        type: varchar(200)
        # → "Book_100000" 形式
      created_at:
        type: timestamp
        # → 2024-01-15 09:00:00
      
  # レビューデータ
  reviews:
    records: 50000
    default_timestamp: "2024-01-15 12:00:00"
    columns:
      review_id:
        type: bigint
        start_from: 1000000
      book_id:
        type: bigint
        start_from: 100000  # 書籍データと整合
      rating:
        type: integer
        # → 1-5の評価
      review_text:
        type: text
        # → "Review text for book..." 形式
      created_at:
        type: timestamp
        # → 2024-01-15 12:00:00
```

## 🎯 PostgreSQL Unnest構文の詳細

UnnestForgeは以下のPostgreSQL機能を最大限活用します：

### 対応データ型
- `INTEGER[]`, `BIGINT[]`, `SMALLINT[]`
- `VARCHAR[]`, `TEXT[]`, `CHAR[]`
- `DATE[]`, `TIMESTAMP[]`, `TIME[]`
- `DECIMAL[]`, `NUMERIC[]`, `REAL[]`
- `BOOLEAN[]`

### バッチサイズ最適化
- デフォルト: 10,000件/バッチ
- 大量データ: 自動的にバッチ分割
- メモリ制限: 設定可能な上限値

## 🤝 コントリビューション

UnnestForgeの改善にご協力ください！

- 🐛 バグレポート
- 💡 機能要望
- 🔧 プルリクエスト
- 📖 ドキュメント改善

## 📄 ライセンス

Apache License 2.0

```
Copyright 2024 UnnestForge Contributors

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

---

**UnnestForge** - Forge Bulk Data with PostgreSQL Unnest Power ⚡

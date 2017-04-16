# 3 HBase Hacks

- HBase Hadoop上で動作するKVS
- 一貫性/整合性/ネットワーク分断耐性を選択したCP型

## 26 Bulkロードツール

- ファイルアップロード
    - tsvファイル
    - csvファイル
- クライアントAPI
- TableOutputFormat
- Bulkロードツール
    - importtsv

- 標準で用意されているツールを使って容易にHBaseにデータをロードできる

## 27 MySQLからのインポート

- Sqoopを使うと便利
    - カスタムPutTransformer

## 28 HFileへ直接アクセスをするMapReduce

- HFileOutputFormatを使ってBulkロード用のHFileを作る方法を紹介
    - タイムスタンプの指定ができないなど細かい調整ができない
    - その代替としてのHFileOutputFormat

## 29 pre-splitテーブルの作成

- 予めリージョンを分割したテーブルを作成
    - 自動的にテーブルをリージョン単位に分けてシャーディングを実現するが、処理が重い
    - ので、pre-splitで予めリージョンを分割しておく
        - JavaAPI
        - HBase Shell

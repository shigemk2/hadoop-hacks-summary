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

## 30 Coprocessorの作り方

- HBase0.92
    - クライアントサイドで行っていた処理をサーバーサイドで行えるようにするための仕組み
    - Observer
        - トリガー
    - Endpoint
        - カスタムRPCプロトコルを提供

## 31 カスタムFilterの作り方

- Scam/Getの際にFilterを指定して、取得するデータのフィルタリングが可能
    - サーバーサイドでフィルタリングするので不要なデータ転送が発生しない
    - カスタムFilter
        - RowKey
        - KeyValue
        - KeyValueのリスト

## 32 export/importツール

- export
    - HBaseデータのバックアップを行うための方法の1つ
- import
    - エクスポートしたデータをもとに別のクラスタを構築

## 33 クラスタレプリケーション

- HBaseは2つのクラスタ間でのデータのレプリケーション機能を提供
- MySQLのmaster/slaveレプリケーションと同じ仕組み
    - WALに書き込まれているPut/Deleteなどのデータを利用する
    - マスタークラスタのHLogを読み込んでテーブルごとにバッファリングして、HBaseクライアントAPIを使ってスレーブクラスタへ書き込む
    - レプリケーションの進捗はZooKeeperによって管理される

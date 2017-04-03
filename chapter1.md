# 1 システム構築/運用Hacks

- Hadoopを使いこなす上でインフラ基盤の観点から重要なポイントをまとめた
- 複数サーバーを協働動作させる「分散システム」
    - 少し特殊なノウハウが必要
- データの特性に応じてRDBMSとHadoopを使い分ける必要がある
- Hadoopはバッチ処理の仕組み

## 1. Hadoop動作に必要なパラメータ

- OSとJava環境がインストールされた環境にHadoopをインストールする
- CDHはRPMやdebパッケージを容易
- /etc/hadoop-0.20/conf

### HDFS

- HDFSマスター
    - fs.default.name
- NameNode用データ格納領域
    - fsimage
    - edits
    - fstime
    - VERSION
- DataNode用データ格納領域

### MapReduce

- MapReduceマスター
    - mapred.job.tracker
- MapReduce環境用ディレクトリ
    - mapred.local.dir

プロパティの設定は必要

## 2 Hadoop用ノードのLinuxOS設定

各ノードにはかなりのCPU ディスクI/O ネットワーク負荷がかかる

### ファイルディスクリプタ

最大ファイルディスクリプタ数の引き上げ

- /etc/security/limits.conf
- root sort nofile 65536

### カーネルパラメータの設定

- /etc/sysctl.confに項目追加
- rootユーザーで`/sbin/sysctl -p`を実行

### ntpdのインストール

一致させておかないとすべてのノードの時刻を合わせておかないとエラーログの解析が困難になる

### Javaのインストール

- 最新のバージョンが推奨
    - 特にv1.6.0_18はNG

## 3 マスターノードのHA化

- Hadoopは大量のサーバーを利用することを前提
- 可用性向上の仕組み

マスターノードをHA化することでSPOFを排除する

### HA構成

- Pacemaker
    - クラスタ制御機能
    - リソース制御機能
- DRDB
    - 2つのノード間でハードディスクなどのブロックデバイスのデータを同期

- サービス自動起動の無効化
- 各設定ファイルの編集
- DRBDのインストールと設定(NameNodeだけ)
    - DRDBインストール
    - DRDBの設定
    - DRDBの起動と停止
    - DRDBの起動確認
    - NameNodeの管理情報格納用ファイルシステムの作成
- Pacemakerのインストールと設定
    - Pacemakerのインストール
    - Pacemakerの設定
    - Pacemakerの起動と停止
- リソース管理の設定
    - Pacemakerが管理するリソースを登録する
    - NameNodeのHAクラスタへのリソース登録
    - JobTrackerのHAクラスタへのリソース登録
    - CIBの変更
    - HAクラスタの状態確認
- フェイルオーバーの確認

- トラブルシューティング
    - DRDB
        - データ同期用LANを切断してしまった時
            - LANの障害を取り除いて`drbdadm connect nn`
        - スプリットブレインの解消
            - 2つのノードがPrimaryロールとなりノードのメタデータで不整合が起きる
            - 更新の破棄
    - Pacemaker
        - Pacemaker停止
        - スプリットブレインの原因除去
        - DRDBの起動とスプリットブレインの解消
        - Pacemaker起動
        - Pacemaker状態確認

## まとめ

- STONITH
- IPMI

# 4 Hadoopに関する統計情報

- クラスタの状態を確認
- サーバーの増設や以上の予兆をつかむ
- そのために統計情報を取得する

## HDFSに関する統計情報
### NameNode自身の統計情報

- ブロック操作に関する情報
- ファイル操作に関する情報
- メタ情報操作に関する情報
- ブロックレポートに関する情報

### HDFSに関する統計情報

- HDFSの容量に関する情報
- HDFS上のブロックに関する情報
- HDFS上のファイル数

### DataNode自身の統計情報

- ブロック操作に関する情報
- クライアントとのやりとりに関する情報
- ハートビートに関する情報

## MapReduceに関する統計情報
### JobTracker自身に関する統計情報

- MapReduceジョブに関する情報
- タスクに関する情報
- 管理しているTaskTrackerに関する情報

### TaskTracker自身に関する統計情報

- スロットに関する情報
- タスクに関する情報

### Shuffleに関する統計情報

- Shuffleフェーズでの操作に関する統計情報を取得

## JVMに関する統計情報

- ヒープメモリに関する情報
- GCに関する情報
- ログ出力に関する情報
- スレッドに関する情報

## RPCに関する統計情報

- 通信に関する情報
- 認証 認可に関する情報
- 共通で取得できる情報
- NameNodeのみ取得できる情報
- JobTrackerのみ取得できる情報

## 統計情報の取得手段

以下の4つ。

- ファイルに統計情報を出力
- JMX経由で出力
- Ganglia経由で出力
- HadoopのWebインターフェイス経由で出力

## まとめ

Hadoopクラスタの統計情報を取得する理由はHadoopクラスタが正常に動作しているかどうかやHadoopクラスタの増設やチューニングなど運用面で指標を確認するため

# 5 HDFSのアップグレード

HDFSをアップグレードする場合の手法について、CDHをベースに説明する

Hadoopの利用開始後にアップデートがあることがままある

## CDH3の同一バージョンの更新について

+ Hadoopクラスタのすべてのノードを停止
+ 新規パッケージ(RPM/deb)をすべてのノードに配布
+ すべてのノードでHadoopパッケージを更新(upgrade)
+ Hadoopクラスタのすべてのノードを起動

## 異なるCDHバージョン間での更新について

+ Hadoopクラスタのすべてのノードを停止
+ 新規パッケージ(RPM/deb)をすべてのノードに配布
+ すべてのノードでHadoopパッケージを更新(upgrade)
+ NameNodeをアップグレードコマンドにて起動
+ Hadoopクラスタのその他のノードを起動

## まとめ

頻繁にHDFSをアップグレードするケースは少ないがこの内容をもとに方針を決めること

# 6 Sqoopの構造と動作

RDBMSからHDFSへのデータインポート、HDFSからRDBMSへのデータエクスポートを行うために利用する

## 準備

`sudo yum install sqoop`

## 動きを理解する
### インポート

RDMBS→HDFS

+ テーブル定義情報の取得
+ 定義情報にしたがってselect分を発行するMapperコードを自動生成
+ 同コードをjarにかためてHadoopフレームワークにて実行
+ データをDBより並列で取得してHDFSに書き出し

### エクスポート

HDFS→RDMBS

+ テーブル定義情報の確認(当該DBにSQLを発行)
+ 定義情報にしたがってデータを書き出すinsert文を発行するMapperコードを自動生成
+ 同コードをjarにかためてHadoopフレームワークにて実行
+ データをHDFSより並列でDBに書き出し

## Oracleを使った動作確認

以下を考慮すること

- インポートするテーブル名は小文字ではなく大文字で書く
- スキーマ名.Table名のようにスキーマ名を指定してはいけない

### OraOopのインストール

OracleとHadoop間のデータ移動を高速化させるツールとしてOraOopがある

+ OraOopをダウンロード
+ oraoop.tarを展開
+ SQLLP_HOMEを設定
+ インストール

各種動作確認

## まとめ

- Sqoopの概要説明
- 単体使用時のOracleに対する挙動
- OraOop導入時のOracleに対する挙動
- Oracle⇔Hadoopの際にOraOopを導入し並列で実行するのが有効
- OraOopはSqoopのcodegenには対応していない
    - テーブル名は大文字で書いて、スキーマ名を指定しないということを念頭に置く

# 7 PostgreSQLでの動作
## SqoopのPostgreSQL連携機能

- PostgreSQLからHDFSへのSQLによるインポート
- PostgreSQLからHDFSへダイレクトインポート(psql)
- HDFSからPostgreSQLへのSQLによるエクスポート

## PostgreSQLでの利用

- PostgreSQL JDBC
- PostgreSQLパッケージ

## PostgreSQLによるダイレクトインポート

- psqlを利用したRDBMS上のデータをHDFSに格納する

## 連携の問題

- ダイレクトエクスポート機能がない
- ダイレクトインポートのクライアントの非スケール

# 8 Azkaban入門

LinkedInによって開発されたシンプルなジョブスケジューラー

1. スタンドアローンデプロイ(同梱jettyで動かす)
2. Tomcatデプロイ

## インストール

- ダウンロード
- インストール
- 環境の設定
- 実行

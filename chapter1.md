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

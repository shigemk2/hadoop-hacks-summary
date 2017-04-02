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

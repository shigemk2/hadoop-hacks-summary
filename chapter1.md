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

### まとめ

- STONITH
- IPMI


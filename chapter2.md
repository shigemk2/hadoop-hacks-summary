# 2 アプリケーション開発Hacks

Hadoopをクラスタ内部からではなく外部のプロキシサーバーなどから操作する方法

Hadoopの操作を実行するためにはhadopコマンドを使う

## 10 クラスタ外部からHadoopの操作

- 外部のHadoopに設定ファイルをコピーする($HADOOP_$HADOOP_HOME/conf)
    - core-site.xml
    - hdfs-site.xml
    - mapred-site.xml
    - これによりhadoopコマンドが使える
- アプリケーションからMapReduceを操作する
    - ToolRunner.runメソッドを使う

## 11 InMapperCombiner

- MapReduceジョブの中間データを削減して通信のオーバーヘッドを緩和するテクニックが **InMapperCombiner**
    - Reduceタスクで行う処理の一部をShuffleの直前で行うことで中間データ量を削減し、shuffle字の負荷を削減する仕組み
    - InMapperCombinerのテクニックでユーザーが独自にCombinerの仕組みを実装できる

### InMapperCombiner

通常のCombinerと比較した利点

+ Combinerの実行契機や実行方法を制御できる
+ Shuffleフェーズに流れるデータ量の削減だけでなくMapタスクから出力されるデータが無駄にインスタンス化されるのを防ぐ

- インスタンス化にはコストが発生する

### 実装にあたって抑えておくべきポイント

+ HashMapなど連想配列をカスタムMapperクラスのメンバーに用意する
+ Mapタスクの出力はmapメソッドでの処理の都度Contextクラスのインスタンスメソッドwriteメソッドですぐに中間データとして出力するのではなく、連想配列を利用して集約処理/格納しておいて、後でまとめて連想配列の中身を書き出す
+ 連想配列のサイズの拡張に伴いヒープを圧迫しないよう、一定数、もしくは一定サイズ以上のデータが格納された時点で、中身をwriteメソッドで書き出す
+ カスタムMapperクラスでcleanupメソッドをオーバーライドし、あるMapタスクに割り当てられたすべてのデータを処理し終えたときに、連想配列に残っているすべてのデータをwriteメソッドで出力

## 12 カスタムWritable型の作り方

MapReduceのI/Oに独自のシリアライズ形式を利用したい場合や任意のデータ構造を定義したいときに活用できる

- Writable Hadoopがが扱うのに適したデータ型
    - Mapタスクから中間データを出力する
    - SequenceFileにデータを書き出す
    - データをシリアライズするときに役立つ
- すでに用意されているWritable
    - Textクラス
    - IntWritable
- カスタムWritable
    - Writableインターフェイス
    - WritableComparableインターフェイス

より効率の良いシリアライズ/デシリアライズ機構をもつ複合データ型を定義できる

## 13 カスタムPartitonerの作り方

- Hadoopの処理 Map Shuffle Reduce
    - 通常は中間データのkeyのハッシュ値が同じものを同じReduceタスクに振り分ける
    - 案件によっては任意のルールで振り分けしたい
    - そんな場合にPartitionerクラスをよぶ or 継承する
- カスタムPartitioner
    - Partitionaerクラスを継承したもの
    - ポピュラーな例として複数の軸を基準としてソートを行うセカンダリソートがある

## 14 DistributedCache

- マスターデータなどのようにサイズがあまり大きくなくて、かつ参照のみに利用するデータを複数のMapタスクやReduceタスクから効率的に利用するための仕組み

### DistributedCacheとは

- アイデア1 HDFSにマスターデータを配置し、複数のタスクから参照する方法
    - タスクを起動するたびにHDFSへデータを読み取る必要があるからオーバーヘッドが発生する
- アイデア2 各スレーブサーバーのローカルファイルシステムにあらかじめマスターデータを配布しておく仕組み
    - 作りこまなくてもDistributedCacheが使える
    - ジョブの開始時にHDFS上のマスターデータを自動的にスレーブサーバーのローカルファイルシステムに配布する
    - ユーザーが独自にマスターデータをスレーブサーバーに配布する仕組みを作りこむ手間が省ける
    - シンボリックリンクでユーザーから透過的に利用できる

## 15 CombineFileInputFormat

- 「大量の小さなデータ」を効率的に処理する
- FileInputFormat
- HDFSのブロックサイズより小さい
    - スループットの低下
    - Mapタスクの生成によるタスク制御のオーバーヘッドの増大
- CombineFileInputFormat
    - 複数のファイルをまとめたスプリットを作れる
    - Mapタスクで効率よく処理できる
- 独自のCombineFileInputFormat
    - オリジナルの定義は抽象クラスで、独自実装はサブクラス
    - カスタムRecordReader
- HDFSは大量の小さなファイルを管理するのが苦手
- 一度HDFSにストアした大量のファイルを集約してHDFSのヒープ領域の圧迫を回避

ブロック: HDFSに格納するファイルを一定サイズで分割したデータ http://oss.nttdata.co.jp/hadoop/hadoop.html

## 16 MapReduceジョブをテストする

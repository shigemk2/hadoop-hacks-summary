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

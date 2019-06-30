# はじめに
本記事ではJavaにおける並列処理を主に個人の備忘録、勉強用として雑に記しています。

# 並列処理
並列処理とは複数のスレッドを使って、メインスレッドとは別のスレッドでプロセスを行うことである。

## 並列処理における問題例
スレッドAがメモリを読み込んでいる間にスレッドBがそのメモリに書き込みをしたら、スレッドAで読み取られるのは新しい値か古い値か？

## なぜマルチスレッドプログラミングではなく並列処理なのか
マルチスレッドプログラミングや分散コンピューティングで起きる問題は並列処理と似ているのでここでは並列処理を使います。

# なぜ並列処理するのか
- リソース利用の最適化
    - CPUのアイドルタイムを減らす
- プログラムの高速化
    - 例えばサーバーからリクエストをlistenしてそのリクエストを処理するループがあるとする。
    ```java
      while(server is active){
        // リクエストを受ける
        ...
        // リクエストを処理する
    }
    ```
    このループではリクエストを処理してる間は他のリクエストを受けれなくなってる。もしリクエストの処理のタスクを他のスレッドに受け渡せば、すぐにまたリクエストを受けれるようになる。
    ```java
      while(server is active){
        // リクエストを受ける
        ...
        // リクエストを他スレッドに受け渡す
    }
    ```

# 主な並列処理モデル
ここで紹介する並列処理モデルは分散コンピューティングシステムに使われるモデルと共通するのもあります。

## Parallel Workers
![](images/concurrency-models-1.png) 

出典: [Jenkov.com](http://tutorials.jenkov.com/java-concurrency/concurrency-models.html#concurrency-models-and-distributed-system-similarities)

Delegatorがworkerにタスクを分散する方法です。Parallel worker モデルは最もJavaの並列処理に使われているモデルです。`java.util.concurrent`パッケージに含まれているツールはこのモデルに基づいて作られています。

### 利点/欠点

- 利点
    - 簡単
- 欠点
    - 複数のworkerが同じメモリもしくはデータベースを改変している場合、workerスレッドはその改変が他のworkerスレッドにも知らされるようにならなければいけません。(CPUの実行に留まらず、)
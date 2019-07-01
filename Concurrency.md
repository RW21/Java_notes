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
    - 複数のworkerが同じメモリもしくはデータベースを改変している場合、workerスレッドはその改変が他のworkerスレッドにも知らされるようにならなければいけません。(CPUの実行に留まらず、メモリにプッシュされなければならない)
    - Race conditionsやdeadlockとはいった並列処理におけるよくある問題を避けなければならない。
    -スレッドが同じデータ構造へのアクセスを待ってしまうと並行化(parallelisation)が失われてしまう。
    - タスクの順番が決定的ではない。これにより、現在のシステムのステートがどうなっているのか分からなくなる場合がある。

## Assembly Line

![](images/concurrency-models-3.png)

出典: [Jenkov.com](http://tutorials.jenkov.com/java-concurrency/concurrency-models.html#concurrency-models-and-distributed-system-similarities)

Workerにタスクの役割があって、一つのタスクが終わると次のWorkerにタスクが移行する。

![](images/concurrency-models-6.png)

出典: [Jenkov.com](http://tutorials.jenkov.com/java-concurrency/concurrency-models.html#concurrency-models-and-distributed-system-similarities)

こんなことになることも。

### 利点/欠点

- 利点
    - 共有ステートがない
    - タスクに順番がある
- 欠点

## Same-threading

シングルスレッドのプログラムを複数のCPUコアを活用するために複数コアに渡ってスケールすること。このモデルでは共有されるデータ構造、メモリはありません。

# 並列(concurrency) vs 並行(parallelism)

並列化と並行化という単語は同じマルチスレッドプログラミングにおいてよく使われますが、同一ではありません。

## 並列化

システムが複数のタスクを一度にこなすこと。

## 並行か

タスクがサブタスクに分散して一度に複数のサブタスクをこなすこと。

# Javaでスレッドを作る

Javaではスレッドはオブジェクトとして扱われています。

```java
public class StartThread {

    public static void main(String[] args) {
        Thread myThread = new Thread();
        myThread.start();
    } 
}
```

`myThread.start();`でスレッドを始めています。しかしこれではスレッドは何もしていません。

## `run`メソッドをオーバーライドしてスレッドでコードを動かす

スレッドに動くコードを指定するには`Thread`のサブクラスを作り、`run`メソッドをオーバーライドします。

```java
public class StartThread {

    public static class MyThread extends Thread {
        @Override
        public void run() {
            System.out.println("Hello World");
        }
    }

    public static void main(String[] args) {
        Thread myThread = new MyThread();

        myThread.start();
    }

}
```

## `Runnable`インターフェースでスレッドを実装

`Runnable`をコンストラクタとして通してスレッドを作ることができます。

```java
public class StartThread {

    public static class MyRunnable implements Runnable {
        @Override
        public void run() {
            System.out.println("Hello again");
        }
    }

    public static void main(String[] args) {
        Thread myThread1 = new Thread(new MyRunnable());

        myThread1.start();
    }

}
```
# はじめに

本記事ではJavaにおける並列処理を主に個人の備忘録、勉強用として雑に記しています。気が向いたら更新します。主にJavaの並列化に使うキーワードを紹介しています。

# 並列処理

並列処理とは複数のスレッドを使って、メインスレッドとは別のスレッドでプロセスを行うことである。

## 並列処理における問題例

スレッドAがメモリを読み込んでいる間にスレッドBがそのメモリに書き込みをしたら、スレッドAで読み取られるのは新しい値か古い値か?


## 並列(concurrency) vs 並行(parallelism)

並列化と並行化という単語は同じマルチスレッドプログラミングにおいてよく使われますが、同一ではありません。

### 並列化

システムが複数のタスクを一度にこなすこと。

### 並行化

タスクがサブタスクに分散して一度に複数のサブタスクをこなすこと。

### マルチスレッドプログラミング? 分散コンピューティング? 並行処理? 並列処理?

マルチスレッドプログラミングや分散コンピューティングの概念は並列処理の概念とよく似ています。基本的には(並行処理, 並列処理)⊆マルチスレッディングです。本記事では並列処理しか扱いません。


# なぜ並列処理するのか

- リソース利用の最適化
    - CPUのアイドルタイムを減らす。
- プログラムの高速化
    - 例えばサーバーからリクエストをlistenしてそのリクエストを処理するループがあるとする。
    ```java
      while(server is active){
        // リクエストを受ける
        ...
        // リクエストを処理する
    }
    ```
    このループではリクエストを処理してる間は他のリクエストを受けれなくなっている。もしリクエストの処理のタスクを他のスレッドに受け渡せば、すぐにまたリクエストを受けれるようになり、高速化できる。
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
    - 複数のworkerが同じメモリもしくはデータベースを改変している場合、workerスレッドはその改変が他のworkerスレッドにも知らされるようにしなければならない。(CPUの実行に留まらず、メモリにプッシュされなければならない)
    - [競合状態](#競合状態とクリティカルセクション)やデッドロックといった並列処理におけるよくある問題を避けなければならない。
    -スレッドが同じデータ構造へのアクセスを待ってしまうと並行化(parallelisation)が欠けてします。
    - タスクの順番が決定的ではない。これにより、現在のシステムのステートがどうなっているのか分からなくなる場合がある。

## Assembly Line

![](images/concurrency-models-3.png)

出典: [Jenkov.com](http://tutorials.jenkov.com/java-concurrency/concurrency-models.html#concurrency-models-and-distributed-system-similarities)

Workerにタスクの役割があって、一つのタスクが終わると次のWorkerにタスクが移行する。

![](images/concurrency-models-6.png)

出典: [Jenkov.com](http://tutorials.jenkov.com/java-concurrency/concurrency-models.html#concurrency-models-and-distributed-system-similarities)

上の図のようになることも。

### 利点/欠点

- 利点
    - 共有ステートがない。
    - タスクに順番がある。
- 欠点

## Same-threading

シングルスレッドのプログラムを複数のCPUコアを活用するために複数コアに渡ってスケールすること。このモデルでは共有されるデータ構造、メモリはありません。

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

## スレッドが取り得る状態

- New
    - スレッドが作られて、まだ実行(`run`)されていない状態。
- Runnable
    - スレッドが実行された状態。
- Blocked
    - 同期ブロックを実行待ち。(他のスレッドがすでに実行している。)
- Waiting
    - 他のスレッドを待っている。[スレッドシグナリング](#スレッドシグナリング)でWaitingになったり解除されたりする。

# 競合状態とクリティカルセクション

競合状態はクリティカルセクションで起こりうる状態です。

## クリティカルセクション

クリティカルセクションとはコードが複数のスレッドで実行されて、その順番によって結果が異なる状態です。複数のスレッドが同じクリティカルセクションに書き込みしようとすると問題が発生します。

```java
  public class Counter {

     protected long count = 0;

     public void add(long value){
         this.count = this.count + value;
     }
  }
```

上記のクラスでは`add`メソッドがクリティカルセクションになります。

## 競合状態

上のクリティカルセクションを2つのスレッドAとBで実行されます。まずスレッドAとBがレジスターに`this.count = 0`を読み込みます。まずスレッドAが読み込まれた`this.count`に2を足します。その後、スレッドBが読み込まれた`this.count`に3を足します。この場合、最終的な結果は`this.count = 5`ではなく、`this.count = 3`になります。

上記のように複数のスレッドが同じリソースを取り合うことを競合状態といいます。

## 競合状態を防ぐ

競合状態を防ぐ一つの方法にはクリティカルセクションを不可分操作(atomic)状態にするというのがあります。つまり、一つのスレッドが実行中には他のスレッドが実行できない状態にします。

これを実現するにはクリティカルセクションでスレッド同期を行います。スレッド同期を行うこと以外ではスレッドロックや`java.util.concurrent.atomic.AtomicInteger`などの不可分操作変数を使うことができます。

# スレッドセーフと共有リソース

複数のスレッドに同時で実行されても安全なコードをスレッドセーフと呼びます。スレッドセーフなコードは競合状態を起こしません。競合状態は複数のスレッドが共有リソースをアクセスすることで起こります、なのでどういったリソースが共有なのかを知っておくことが大切です。

## ローカル変数

ローカル変数はスレッドのスタックに保存されるのでスレッド間で共有されることがありません。プリミティブ型のローカル変数は常にスレッドセーフです。

## ローカルオブジェクト参照

この場合は、参照自体は共有されません。しかしオブジェクトは共有ヒープに保存されます。もしオブジェクトがローカルで作成されメソッドの外には出ない場合、スレッドセーフです。

## メンバ変数

メンバ変数はオブジェクトと共にヒープに保存されています。スレッドセーフではありません。

# `synchronized`キーワードで同期ブロック

Javaの同期ブロックはすべて`synchronized`のメソッド修飾子で実装できます。

```java
  public synchronized void add(int value){
      this.count += value;
  }
  ```

メソッド全体を同期ブロックにするのが好ましくない場合は、一部だけ同期ブロックにすることができます。

```java
  public void add(int value){

    synchronized(this){
       this.count += value;   
    }
  }
```

同期ブロックの中のコードは複数のスレッドで同時に実行されることはありません。

# `volatile`キーワード

Volatileとは揮発性という意味です。プログラミングにおいては値が変更されていないようにみえてもアクセスする度に変わっているという意味になります。Javaの`volatile`キーワードは変数がCPUキャッシュにではなくメインメモリに保存されていることを保証をします。

例えば以下のような共有されている変数があるとします。

```java
public class SharedObject {

    public int counter = 0;

}
```

スレッド1は`counter`をインクリメントします。スレッド1とスレッド2は度々`counter`を読みます。もし`counter`が`volatile`だと宣言されていない場合は、メインメモリからではなく、CPUキャッシュから値が読まれる恐れがあります。以下の図がこのシチューションを表しています。

![](images/java-volatile-2.png)

出典: [Jenkov.com](http://tutorials.jenkov.com/java-concurrency/volatile.html)


## `volatile`だけでは十分じゃない場合

スレッドが`volatile`な変数を読み込んで、それを元に新しい値に更新してしまうと、その変数は固定されるのを保証されません。変数を読み込んで書き込むまでの間に他のスレッドが書き込む可能性があり、競合状態が発生します。

# `ThreadLocal`

`ThreadLocal`クラスは一つのスレッドにしか読み書きできない変数を作るのに使います。もし2つのスレッドが同じ`ThreadLocal`の参照を読み込むと2つのスレッドは互いの`ThreadLocal`変数にアクセスできません。

## `ThreadLocal`変数の作成

```java
private ThreadLocal myThreadLocal = new ThreadLocal();
```

## `ThreadLocal`変数へのアクセス

`ThreadLocal`変数が作成されると以下のように値を設定できます。

```java
myThreadLocal.set("A thread local value");
```

値を読み込むには:

```java
String threadLocalValue = (String) myThreadLocal.get();
```

# スレッドシグナリング

スレッドシグナリングでスレッド間にシグナルを送ることができます。シグナリングを使えばスレッドが他のスレッドを待機するということができるようになります。

## 共有オブジェクトでシグナリング

共有変数の値を変更することでスレッド間のシグナリングを行うのが最も簡単です。

```java
public class MySignal{

  protected boolean hasDataToProcess = false;

  public synchronized boolean hasDataToProcess(){
    return this.hasDataToProcess;
  }

  public synchronized void setHasDataToProcess(boolean hasData){
    this.hasDataToProcess = hasData;  
  }

}
```

出典: [Jenkov.com](http://tutorials.jenkov.com/java-concurrency/thread-signaling.html)

## `wait()` `notify()` `notifyAll()`

`java.lang.Object`にはシグナルを待機しているスレッドを非アクティブ化するメカニズムがあります。`wait()`をオブジェクトに呼び出すスレッドは他のスレッドがそのオブジェクトが`notify()`を実行するまで非アクティブになります。

# デッドロック

デッドロックとは本来スレッドシグナリングで進行するはずの複数のスレッドがブロック状態(非アクティブ)になっていて、プログラムが動かない状態を指します。簡単な例をあげると、スレッドAとスレッドBがあって、スレッドAがスレッドBをロックしてスレッドBを待機する、スレッドBがスレッドAをロックしてスレッドAを待機する。

# 生産者消費者問題

並列処理の問題としてよく取り上げられるのが生産者消費者問題です。この問題では、一つのデータ構造から複数の消費者スレッドがアイテムを取り出し、複数の生産者がアイテムを追加します。

## データ構造

データ構造は先に入れられたデータから順に取り出されるキューで実装します。

## スレッド

消費者スレッドはキューがいっぱいになると生産者スレッドをブロックします。(これ以上商品を足すのを防ぐため。)逆に、生産者スレッドはキューが空になると消費者スレッドをブロックするよう実装します。

これらはいちいちスレッドシグナリングを実行しなくても`java.util.concurrent.BlockingQueue`で簡単に実装できてしまいます。

![](images/producer_consumer.jpg)

# 参照

- [Jenkov.com](http://tutorials.jenkov.com/java-concurrency/index.html)
- [Java documentation](https://docs.oracle.com/javase/tutorial/essential/concurrency/)
- [Winterbe](https://winterbe.com/posts/2015/04/07/java8-concurrency-tutorial-thread-executor-examples/)


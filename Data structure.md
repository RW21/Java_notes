# はじめに

Javaの様々なデータ構造の特徴や使い方等を適当に挙げていきます。

# 配列(array)

まずは基本的なデータ構造の配列。

```java
int[] array = new int[5];
// array ==> int[5] { 0, 0, 0, 0, 0 }

String[] array1 = new String[5]
// array1 ==> String[5] { null, null, null, null, null }

// オブジェクト名を取得
array.getClass().getSimpleName()
// ==> "int[]"
```

一見Javaの配列はオブジェクトとしては扱われていないように見えますが、れっきとしたオブジェクトです。

> An object is a class instance or an array.

出典: [javase docs](https://docs.oracle.com/javase/specs/jls/se8/html/jls-4.html#jls-4.3.1)

## 特徴

- 順序を持つ。
- インデックスが早い。
- サイズが固定されている。

## 配列に対するアクションの計算量

配列は早いです。

- Indexing
    - O(1)
- Search 
    - O(n)


# List

Javaではリストはインターフェースです。このインターフェースは`ArrayList`、`LinkedList`、`Vector`、`Stack`で実装できます。

```java
List a = new ArrayList();
List b = new LinkedList();
List c = new Vector(); 
List d = new Stack(); 
```

## 特徴

- 順序を持つ。
- サイズが動的なので挿入ができる。

## Listに対するアクションの計算量

- Indexing
    - O(1)
- Search 
    - O(n)
- Insertion(挿入)
    - O(n) (配列を作り直すため。)

## ArrayList

最も基本的なList。[docs](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html)

```java
List<Integer> l1 = new ArrayList<Integer>(); 
// l1 ==> []

// 挿入が成功するとtrue値を返す
l1.add(4);
// ==> true
```




# 参照

- https://www.geeksforgeeks.org/

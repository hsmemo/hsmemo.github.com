---
layout: default
title: Exception の処理 ： 処理の概要
---
[Up](no2114VSZ.html) [Top](../index.html)

#### Exception の処理 ： 処理の概要

--- 
## 概要(Summary)
例外の送出は, 以下のような場合に行われる.

どの場合にも, 対応する例外ハンドラの探索処理が行われ,
適切なハンドラが見つかるとそれを呼び出すことで捕捉処理が行われる.
見つからなければ, スタックフレームを破棄して一つ上のフレームに遡り, 同じ処理が繰り返される.

  * Java コード中での例外発生 (例外を起こすようなバイトコード処理での例外発生)
      * athrow の処理で発生する例外
      * その他のバイトコードの処理で発生する例外
          * ランタイム処理中に検出される例外     (メモリ確保処理で OufOfMemoryException, ダイナミックリンク失敗で NoSuchMethodError, 等)
          * 明示的なチェックによって検出する例外  (ArrayIndexOutOfBoundsException, 等)
          * signal handler で検出する例外     (NullPointerException, StackOverflowError, ArithmeticException) (備考も参照)

  * JNI コード内(ネイティブメソッド内)での例外発生 (別項参照 (See: [here](nok1NPdCrM.html) for details))

## 備考(Notes)
以下の例外については, 明示的に検出する代わりに
OS にチェックさせて signal handler (See: [here](noNmlmYDJk.html) for details) で受けることもできる.

  * NullPointerException
  * StackOverflowError
  * ArithmeticException

ただし, 何を signal handler で検出するかは環境等によって異なる.

  * 例えば, C++ interpreter では全て明示的にチェックしている模様.

```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    #ifdef CC_INTERP
        // C++ interpreter doesn't throw implicit exceptions
```


## 処理の流れ (概要)
基本的にはどの場合も以下のような流れになる.
ただし, "2." と "3." はまとめて行われることもある.
また, athrow の場合はいきなり "3." から始まる (バイトコードを実行する時点で "1." と "2." は終わっているようなものなので).

1. 例外送出条件の検出処理

   以下の 3つの検出パターンがある.
   行われる処理自体はインタープリタ種別や JIT 種別に応じて異なる.
   
   1. ランタイム処理内での検出      (メモリ確保処理で OufOfMemoryException, ダイナミックリンク失敗で NoSuchMethodError, 等)
   2. 明示的に検出                (ArrayIndexOutOfBoundsException, 等)
   3. signal handler による検出  (NullPointerException, StackOverflowError, ArithmeticException)
   
2. 例外オブジェクトの生成処理

   ランタイム内の生成関数が呼び出される
   (直接ランタイムを呼ばないパスもあるが, 最終的にはどの場合でもランタイム内の生成関数が呼ばれる).

   行われる処理自体はインタープリタ種別や JIT 種別に応じて異なる.

3. 例外の送出処理

   "2."とまとめて行われることが多い.
   その場合はランタイム内からの送出となり, "4." の処理はランタイムフレームの処理から始まる.

   athrow のように直接送出する場合は, ランタイムフレーム以外のフレームから始まることもある.

   行われる処理自体はインタープリタ種別や JIT 種別に応じて異なる.

4. 例外ハンドラの探索処理

   各スタックフレームに応じた方法で例外ハンドラを探す.
   現在のフレーム中で見つからなければ, そのスタックフレームをたたみ, 一つ上のフレームにさかのぼって処理が繰り返される.

   行われる処理自体はインタープリタ種別や JIT 種別に応じて異なる.




## Subcategories
* [Exception の処理 ： 処理の概要 ： (補足) ランタイム内での例外の扱われ方 ](no3059qHd.html)




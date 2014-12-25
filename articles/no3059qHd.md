---
layout: default
title: Exception の処理 ： 処理の概要 ： (補足) ランタイム内での例外の扱われ方 
---
[Up](nogMJcsk14.html) [Top](../index.html)

#### Exception の処理 ： 処理の概要 ： (補足) ランタイム内での例外の扱われ方 

--- 
## 概要(Summary)
HotSpot 内部での処理中に例外送出条件が満たされた場合, 
ネイティブコードのスタックを (C++の例外機能等を用いて) unwind すると処理がコンパイラ依存になったり性能が予見できなくなる恐れがある.
このため, JNI での例外管理に類似した独自のやり方が用いられる.


```cpp
    ((cite: hotspot/src/share/vm/utilities/exceptions.hpp))
    // This file provides the basic support for exception handling in the VM.
    // Note: We do not use C++ exceptions to avoid compiler dependencies and
    // unpredictable performance.
    //
    // Scheme: Exceptions are stored with the thread. There is never more
    // than one pending exception per thread. All functions that can throw
    // an exception carry a THREAD argument (usually the last argument and
    // declared with the TRAPS macro). Throwing an exception means setting
    // a pending exception in the thread. Upon return from a function that
    // can throw an exception, we must check if an exception is pending.
    // The CHECK macros do this in a convenient way. Carrying around the
    // thread provides also convenient access to it (e.g. for Handle
    // creation, w/o the need for recomputation).
```

具体的には, ランタイム内で例外発生に相当する事態が起こると以下のように処理が行われる.

1. 対応する例外オブジェクトが生成された後, その事態を引き起こした Thread オブジェクトの
   pending_exception フィールドにその例外オブジェクトが格納される.

   (ここで生成される例外オブジェクトとは Java レベルのいわゆる「例外オブジェクト」.
   つまり java.lang.Throwable(のサブクラス) のインスタンス.
   これがそのまま Java レベルの例外ハンドラに渡されることになる)

   (なお, より正確に言うと「ThreadShadow クラス」の pending_exception フィールド.
   See: ThreadShadow)

2. (例外発生に相当する事態ということは, その関数での残りの処理は行う必要はないので) その関数からは return する.

3. HotSpot 内では, 例外が起こりうる関数(= pending_exception をセットしうる関数)を呼んだ際には
   必ず呼び出し元が pending_exception の確認を行うことになっている.
   もし pending_exception がセットされていれば, 呼び出し元も後続の処理は中止して return する.

   (ただし, 後始末などを行う必要がある場合は少し処理した後で return したりと, 柔軟に処理できるようになっている)

4. 以上のようにして, 明示的な pending_exception のチェックと return により,
   実際に捕捉される箇所まで例外オブジェクトが持ち上げられていく.

   (ただし, 基本的にランタイム内には例外を捕捉する箇所はない (ランタイムが勝手に例外を握りつぶしても困るし...) ので,
   例外が捕捉されるのはランタイムの関数フレームを全て巻き戻し,
   Java レベルの関数フレームに到達してからになる)








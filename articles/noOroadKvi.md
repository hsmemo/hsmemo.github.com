---
layout: default
title: 同期排他処理 ： ロック確保処理 (monitorenter バイトコード, synchronized メソッド)
---
[Up](no2114NIs.html) [Top](../index.html)

#### 同期排他処理 ： ロック確保処理 (monitorenter バイトコード, synchronized メソッド)

--- 
## 概要(Summary)
ロックの確保処理は, 以下の箇所で行われる.

  * monitorenter バイトコード命令

  * synchronized メソッドの先頭部分(エントリ部分)

    (こちらについては, さらに native method の場合とそうでない場合に分けられる)

どの場合にも, 確保処理には高速な fast-path と低速な slow-path がある.
まず fast-path が試され, それが失敗すると slow-path の処理が行われる.

どの fast-path においても,
「BasicObjectLock を確保して oop を書き込んだ後, 最終的には lock_object() でロック確保を行う」,
という部分は共通.
さらに, slow path の ObjectSynchronizer に入ってからは, cpu 依存(x86,sparc) もなくなり全パターンの処理が共通になる.

## 備考(Notes)
なお JNI 等でのロック処理については別項参照 (See: [here](no5248b4E.html) for details).

## 処理の流れ (概要)
基本的にはどの場合も以下のような流れになる.
"1." の fast-path が失敗した場合にのみ, "2." 以降の slow-path 処理が行われる.

1. fast-path   (See: [here](no7-p6WVd3.html) and [here](noCvhNUCUL.html) for details)

   fast-path での確保処理が行われる.
   行われる処理自体はインタープリタ種別や JIT 種別に応じて異なる. また CPU に応じても異なる.
   
   確保に失敗した場合は slow-path 1 にフォールバックする.

2. slow-path 1  (See: [here](no7zlkLkfb.html) for details)

   各種の Runtime クラスから slow-path 用の確保関数が呼び出される.
   呼び出される関数はインタープリタ種別や JIT 種別に応じて異なる
   (e.g. InterpreterRuntime::monitorenter(), ...).

   確保に失敗した場合は slow-path 2 にフォールバックする.

3. slow-path 2   (See: [here](no96623Ns.html) for details)

   以上の処理で確保に成功しなかった場合,
   どの処理パスも最終的には 
   ObjectSynchronizer::slow_enter() または ObjectSynchronizer::fast_enter() に到達する.
   
   この ObjectSynchronizer::slow_enter()/ObjectSynchronizer::fast_enter() の処理は, 
   インタープリタ種別/JIT 種別/CPU 等によらず共通.
   この中で (ロックが解放されるまで待機処理も行いつつ) ロックの確保処理が完了される.




## Subcategories
* [同期排他処理 ： ロック確保処理 ： fast-path の処理 ： monitorenter 命令の処理](no7-p6WVd3.html)
* [同期排他処理 ： ロック確保処理 ： fast-path の処理 ： synchronized method のエントリ部の処理](noCvhNUCUL.html)
* [同期排他処理 ： ロック確保処理 ： slow-path の処理 (1)](no7zlkLkfb.html)
* [同期排他処理 ： ロック確保処理 ： slow-path の処理 (2)  ](no96623Ns.html)




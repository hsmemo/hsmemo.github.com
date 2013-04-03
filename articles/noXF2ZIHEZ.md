---
layout: default
title: 同期排他処理 ： ロック解放処理 (monitorexit バイトコード, synchronized メソッド)
---
[Up](no2114NIs.html) [Top](../index.html)

#### 同期排他処理 ： ロック解放処理 (monitorexit バイトコード, synchronized メソッド)

--- 
## 概要(Summary)
ロックの解放処理は, 以下の箇所で行われる.

  * monitorexit バイトコード命令

  * synchronized メソッドから脱出する箇所

    (こちらについてはさらに, return による脱出, 例外による脱出, JVMTI の PopFrame/ForceEarlyReturn による脱出, に分けられる.
     また return による脱出の場合は native method の場合とそれ以外で違いがある)

どの場合にも, 解放処理には高速な fast-path と低速な slow-path がある.
まず fast-path が試され, それが失敗すると slow-path の処理が行われる.

どの fast-path においても,
「最終的には InterpreterMacroAssembler::unlock_object() でロック開放を行う」,
という部分は共通.
さらに, slow path の ObjectSynchronizer に入ってからは, cpu 依存(x86,sparc) もなくなり, 全パターンの処理が共通になる.

## 備考(Notes)
なお JNI 等でのロック処理については別項参照 (See: [here](no5248b4E.html) for details).

## 処理の流れ (概要)
基本的にはどの場合も以下のような流れになる.
"1." の fast-path が失敗した場合にのみ, "2." 以降の slow-path 処理が行われる.

1. fast-path (See: [here](noQFS71luo.html), [here](noW9zIplom.html), [here](noA8GKT6vy.html) and [here](nom8Tnx-bW.html) for details)

   fast-path での解放処理が行われる.
   行われる処理自体はインタープリタ種別や JIT 種別に応じて異なる. また CPU に応じても異なる.
   
   確保に失敗した場合は slow-path 1 にフォールバックする.

2. slow-path 1 (See: [here](noqn8CuSLG.html) for details)

   各種の Runtime クラスから slow-path 用の解放関数が呼び出される.
   呼び出される関数はインタープリタ種別や JIT 種別に応じて異なる
   (e.g. InterpreterRuntime::monitorexit(), ...).

   解放に失敗した場合は slow-path 2 にフォールバックする.

3. slow-path 2 (See: [here](noS3vRzujM.html) for details)

   以上の処理で解放に成功しなかった場合,
   どの処理パスも最終的には 
   ObjectSynchronizer::slow_exit() または ObjectSynchronizer::fast_exit() に到達する.
   
   この ObjectSynchronizer::slow_exit()/ObjectSynchronizer::fast_exit() の処理は, 
   インタープリタ種別/JIT 種別/CPU 等によらず共通.
   この中でロックの解放処理が完了される




## Subcategories
* [同期排他処理 ： ロック確保処理 ： fast-path の処理 ： monitorexit 命令の処理](noQFS71luo.html)
* [同期排他処理 ： ロック解放処理 ： fast-path の処理 ： synchronized method の exit 部 ： リターン時](noW9zIplom.html)
* [同期排他処理 ： ロック解放処理 ： fast-path の処理 ： synchronized method の exit 部 ： 例外による unwind 時](noA8GKT6vy.html)
* [同期排他処理 ： ロック解放処理 ： fast-path の処理 ： synchronized method の exit 部 ： JVMTI の PopFrame()/ForceEarlyReturn() 時](nom8Tnx-bW.html)
* [同期排他処理 ： ロック解放処理 ： slow-path の処理 (1)](noqn8CuSLG.html)
* [同期排他処理 ： ロック解放処理 ： slow-path の処理 (2)](noS3vRzujM.html)




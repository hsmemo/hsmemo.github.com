---
layout: default
title: Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： 概要
---
[Up](noadKcOM5n.html) [Top](../index.html)

#### Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： 概要

--- 
## 概要(Summary)
停止処理が行われる処理パスは以下に示すように沢山ある.
ただし停止処理自体は SafepointSynchronize::block() で実装されており, どの場合もこれを呼び出すことで停止する.

停止処理に至る処理パスとしては, 多くの場合, スレッドの状態が遷移する際に停止する
(つまり, 何らかの状態遷移が起こった際に Safepoint 処理が開始されていれば, そのスレッドは停止する).

スレッドの状態は JavaThreadState で表現されており,
以下のクラスがその遷移処理を担当している
(これらのクラスのコンストラクタとデストラクタで遷移処理が行われる).

  * ThreadInVMfromJava
  * ThreadInVMfromUnknown
  * ThreadInVMfromNative
  * ThreadToNativeFromVM
  * ThreadBlockInVM
  * ThreadInVMfromJavaNoAsyncException

実際には, これらのクラスを簡単に使うためのマクロが用意されているため, そのマクロ経由で使われることが多い.
例えば, HotSpot 内に入ってくる関数 (HotSpot 内への entry point になる関数) は,
以下のようなマクロを用いて定義されている.
(HotSpot 内に入ってくる処理は状態遷移処理であり, スレッドの状態は _thread_in_vm に変わる).

  * JRT_ENTRY, JRT_LEAF,
  * IRT_ENTRY, IRT_LEAF,
  * JNI_ENTRY, JNI_LEAF, JNI_QUICK_ENTRY,
  * JVM_ENTRY, JVM_LEAF, JVM_QUICK_ENTRY,

なお, これらのマクロは以下のように分類される.

  * _ENTRY で終わるもの

    thread state を変更するので safepoint になりうる, ということを示す.

    (ThreadInVMfromJava, ThreadInVMfromJava, 等, thread state 変更用のクラスが何種類かこのファイル内で定義されている)


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    // ENTRY routines may lock, GC and throw exceptions
```

  * _LEAF で終わるもの

    safepoint にはならないことを示す (簡単な処理ですぐ抜けるようなものが _LEAF に指定されている模様)

    (<= その中で safepoint になる処理がなければ call 点も safepoint にしなくてよい, ということだと思われる)


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    // LEAF routines do not lock, GC or throw exceptions
```

  * _QUICK_ENTRY で終わるもの

    (#Under Construction)


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    // QUICK_ENTRY routines behave like ENTRY but without a handle mark
```








---
layout: default
title: CollectedHeap クラス関連のクラス (CollectedHeap, GCCauseSetter)
---
[Top](../index.html)

#### CollectedHeap クラス関連のクラス (CollectedHeap, GCCauseSetter)

これらは, Java ヒープ領域を管理するためのクラス (See: [here](no3718kvd.html) for details).

Java ヒープ領域を管理するクラスは使用する GC アルゴリズムによって異なるが,
これらはそういったクラス群の基底クラス.
(See: SharedHeap, GenCollectedHeap, G1CollectedHeap, ParallelScavengeHeap)


### クラス一覧(class list)

  * [CollectedHeap](#nonZa1EjZd)
  * [GCCauseSetter](#no9fiUfvmO)


---
## <a name="nonZa1EjZd" id="nonZa1EjZd">CollectedHeap</a>

### 概要(Summary)
Java ヒープ領域を管理するクラスの基底クラス.

Java ヒープ領域は, CollectedHeap クラスのサブクラスによって管理される.


```
    ((cite: hotspot/src/share/vm/gc_interface/collectedHeap.hpp))
    // A "CollectedHeap" is an implementation of a java heap for HotSpot.  This
    // is an abstract class: there may be many different kinds of heaps.  This
    // class defines the functions that a heap must implement, and contains
    // infrastructure common to all heaps.
```


```
    ((cite: hotspot/src/share/vm/gc_interface/collectedHeap.hpp))
    class CollectedHeap : public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

現状では以下のようなサブクラスが存在する.

* SharedHeap           (abstract class)
  * GenCollectedHeap   (下記以外用)
  * G1CollectedHeap    (G1GC 用)
* ParallelScavengeHeap (ParallelScavenge 用)


```
    ((cite: hotspot/src/share/vm/gc_interface/collectedHeap.hpp))
    //
    // CollectedHeap
    //   SharedHeap
    //     GenCollectedHeap
    //     G1CollectedHeap
    //   ParallelScavengeHeap
    //
```




### 詳細(Details)
See: [here](../doxygen/classCollectedHeap.html) for details

---
## <a name="no9fiUfvmO" id="no9fiUfvmO">GCCauseSetter</a>

### 概要(Summary)
CollectedHeap クラス用のユーティリティ・クラス.

ソースコード中のあるスコープの間だけ, 
GCCause を変更する (= CollectedHeap に GCCause を設定する) ための一時オブジェクト(StackObjクラス).


```
    ((cite: hotspot/src/share/vm/gc_interface/collectedHeap.hpp))
    // Class to set and reset the GC cause for a CollectedHeap.
    
    class GCCauseSetter : StackObj {
```

### 使われ方(Usage)
使う際には, コンストラクタに CollectedHeap と GCCause::Cause を渡す
(コンストラクタ内で現在の GCCause::Cause を待避してから新しい GCCause::Cause を設定し,
 デストラクタ内で以前の GCCause::Cause に戻している).




### 詳細(Details)
See: [here](../doxygen/classGCCauseSetter.html) for details

---

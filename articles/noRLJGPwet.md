---
layout: default
title: GenCollectedHeap クラス関連のクラス (GenCollectedHeap, GenCollectedHeap::GenClosure, 及びそれらの補助クラス(GenPrepareForVerifyClosure, GenGCPrologueClosure, GenGCEpilogueClosure, GenGCSaveTopsBeforeGCClosure, GenEnsureParsabilityClosure, GenTimeOfLastGCClosure))
---
[Top](../index.html)

#### GenCollectedHeap クラス関連のクラス (GenCollectedHeap, GenCollectedHeap::GenClosure, 及びそれらの補助クラス(GenPrepareForVerifyClosure, GenGCPrologueClosure, GenGCEpilogueClosure, GenGCSaveTopsBeforeGCClosure, GenEnsureParsabilityClosure, GenTimeOfLastGCClosure))

これらは, Java ヒープ領域を管理するためのクラス (See: [here](no3718kvd.html) for details).

Java ヒープ領域を管理するクラスは使用する GC アルゴリズムによって異なるが,
これらのクラスは GC アルゴリズムが G1GC でも ParallelScavenge でもない場合に使用される.
(See: G1CollectedHeap, ParallelScavengeHeap)



### クラス一覧(class list)

  * [GenCollectedHeap](#no7Tsq8G2p)
  * [GenCollectedHeap::GenClosure](#noVzLI6IXl)
  * [GenPrepareForVerifyClosure](#no97nKkYd1)
  * [GenGCPrologueClosure](#no3RMNCwjW)
  * [GenGCEpilogueClosure](#no7DQuBK1q)
  * [GenGCSaveTopsBeforeGCClosure](#noYApafGth)
  * [GenEnsureParsabilityClosure](#no4PgZATVr)
  * [GenTimeOfLastGCClosure](#no6j6gn8Q5)


---
## <a name="no7Tsq8G2p" id="no7Tsq8G2p">GenCollectedHeap</a>

### 概要(Summary)
Java ヒープ領域の管理を担当するクラス(CollectedHeapクラス)の1つ (See: [here](no3718kvd.html) for details).

このクラスは, GC アルゴリズムが Serial, ParNew, Serial Old, CMS の場合用.


```cpp
    ((cite: hotspot/src/share/vm/memory/genCollectedHeap.hpp))
    // A "GenCollectedHeap" is a SharedHeap that uses generational
    // collection.  It is represented with a sequence of Generation's.
    class GenCollectedHeap : public SharedHeap {
```




### 詳細(Details)
See: [here](../doxygen/classGenCollectedHeap.html) for details

---
## <a name="noVzLI6IXl" id="noVzLI6IXl">GenCollectedHeap::GenClosure</a>

### 概要(Summary)
GenCollectedHeap 内の Generation オブジェクトを処理する Closure クラス (の基底クラス).


```cpp
    ((cite: hotspot/src/share/vm/memory/genCollectedHeap.hpp))
      class GenClosure : public StackObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/memory/genCollectedHeap.hpp))
        virtual void do_generation(Generation* gen) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classGenCollectedHeap_1_1GenClosure.html) for details

---
## <a name="no97nKkYd1" id="no97nKkYd1">GenPrepareForVerifyClosure</a>

### 概要(Summary)
トラブルシューティング用のクラス (関連する diagnostic オプションが指定されている場合にのみ使用される)
(See: VerifyBeforeGC, VerifyDuringGC, VerifyGCLevel, VerifyGCStartAt).

GenCollectedHeap クラス内で使用される補助クラス (Closure クラス).
指定された Generation オブジェクトに対して
Generation::prepare_for_verify() を呼び出すことで, 検証処理の前準備を行う.


```cpp
    ((cite: hotspot/src/share/vm/memory/genCollectedHeap.cpp))
    class GenPrepareForVerifyClosure: public GenCollectedHeap::GenClosure {
```

### 使われ方(Usage)
GenCollectedHeap::prepare_for_verify() 内で(のみ)使用されている

(GenCollectedHeap::prepare_for_verify() は verify 処理で使用される関数.
実際の verify 処理は Universe::verify() が行うが, その直前でこの関数が呼び出されて前準備を行う.
行う処理としては, GenCollectedHeap 内の全ての Generation オブジェクトに対して
Generation::prepare_for_verify() を呼び出すだけ) (See: [here](no28916sKh.html) for details)




### 詳細(Details)
See: [here](../doxygen/classGenPrepareForVerifyClosure.html) for details

---
## <a name="no3RMNCwjW" id="no3RMNCwjW">GenGCPrologueClosure</a>

### 概要(Summary)
GenCollectedHeap クラス内で使用される補助クラス (Closure クラス).

指定された Generation オブジェクトに対して
Generation::gc_prologue() を呼び出すことで, GC の前準備を行う.
(See: [here](no28916sKh.html) for details)


```cpp
    ((cite: hotspot/src/share/vm/memory/genCollectedHeap.cpp))
    class GenGCPrologueClosure: public GenCollectedHeap::GenClosure {
```

### 使われ方(Usage)
GenCollectedHeap::gc_prologue() 内で(のみ)使用されている
(GenCollectedHeap 内の全ての Generation オブジェクトに対して
Generation::gc_prologue() を呼び出す処理).




### 詳細(Details)
See: [here](../doxygen/classGenGCPrologueClosure.html) for details

---
## <a name="no7DQuBK1q" id="no7DQuBK1q">GenGCEpilogueClosure</a>

### 概要(Summary)
GenCollectedHeap クラス内で使用される補助クラス (Closure クラス).

指定された Generation オブジェクトに対して
Generation::gc_epilogue() を呼び出すことで, GC の後始末を行う
(See: [here](no28916sKh.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/memory/genCollectedHeap.cpp))
    class GenGCEpilogueClosure: public GenCollectedHeap::GenClosure {
```

### 使われ方(Usage)
GenCollectedHeap::gc_epilogue() 内で(のみ)使用されている
(GenCollectedHeap 内の全ての Generation オブジェクトに対して
Generation::gc_epilogue() を呼び出す処理).




### 詳細(Details)
See: [here](../doxygen/classGenGCEpilogueClosure.html) for details

---
## <a name="noYApafGth" id="noYApafGth">GenGCSaveTopsBeforeGCClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

GenCollectedHeap クラス内で使用される補助クラス (Closure クラス).
指定された Generation オブジェクトに対して
Generation::record_spaces_top() を呼び出すことで, その時点での使用量の記録を行う.


```cpp
    ((cite: hotspot/src/share/vm/memory/genCollectedHeap.cpp))
    #ifndef PRODUCT
    class GenGCSaveTopsBeforeGCClosure: public GenCollectedHeap::GenClosure {
```

### 使われ方(Usage)
GenCollectedHeap::record_gen_tops_before_GC() 内で(のみ)使用されている
(GenCollectedHeap 内の全ての Generation オブジェクトに対して
Generation::record_gen_tops_before_GC() を呼び出す処理).
(See: [here](no28916sKh.html) for details)

なお, このクラスは (デバッグ時であることに加えて) ZapUnusedHeapArea オプションが指定されている場合にしか使用されない.


```cpp
    ((cite: hotspot/src/share/vm/memory/genCollectedHeap.cpp))
    void GenCollectedHeap::record_gen_tops_before_GC() {
      if (ZapUnusedHeapArea) {
        GenGCSaveTopsBeforeGCClosure blk;
        generation_iterate(&blk, false);  // not old-to-young.
        perm_gen()->record_spaces_top();
      }
    }
```




### 詳細(Details)
See: [here](../doxygen/classGenGCSaveTopsBeforeGCClosure.html) for details

---
## <a name="no4PgZATVr" id="no4PgZATVr">GenEnsureParsabilityClosure</a>

### 概要(Summary)
GenCollectedHeap クラス内で使用される補助クラス (Closure クラス).

指定された Generation オブジェクトに対して
Generation::ensure_parsability() を呼び出すことで, GC 時に内部を処理できる状態にしておく.


```cpp
    ((cite: hotspot/src/share/vm/memory/genCollectedHeap.cpp))
    class GenEnsureParsabilityClosure: public GenCollectedHeap::GenClosure {
```

### 使われ方(Usage)
GenCollectedHeap::ensure_parsability() 内で(のみ)使用されている
(GenCollectedHeap 内の全ての Generation オブジェクトに対して
Generation::record_gen_tops_before_GC() を呼び出す処理)
(See: [here](no28916sKh.html) for details)




### 詳細(Details)
See: [here](../doxygen/classGenEnsureParsabilityClosure.html) for details

---
## <a name="no6j6gn8Q5" id="no6j6gn8Q5">GenTimeOfLastGCClosure</a>

### 概要(Summary)
GenCollectedHeap クラス内で使用される補助クラス (Closure クラス).

指定された Generation オブジェクトに対して
Generation::time_of_last_gc() を呼び出し, 
その値がそれまでに取得した値より小さければ記録する.


```cpp
    ((cite: hotspot/src/share/vm/memory/genCollectedHeap.cpp))
    class GenTimeOfLastGCClosure: public GenCollectedHeap::GenClosure {
```

### 使われ方(Usage)
GenCollectedHeap::millis_since_last_gc() 内で(のみ)使用されている
(GenCollectedHeap 内の全ての Generation オブジェクトに対して
Generation::time_of_last_gc() を呼び出し, それらの値の内で一番小さいものを得るための処理).

(なお, これは sun.misc.GC.maxObjectInspectionAge() メソッドを実現するため(だけ)の関数.
 (See: JVM_MaxObjectInspectionAge()))




### 詳細(Details)
See: [here](../doxygen/classGenTimeOfLastGCClosure.html) for details

---

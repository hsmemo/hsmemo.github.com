---
layout: default
title: VM_GC_Operation クラス関連のクラス (VM_GC_Operation, VM_GC_HeapInspection, VM_GenCollectForAllocation, VM_GenCollectFull, VM_GenCollectForPermanentAllocation, SvcGCMarker)
---
[Top](../index.html)

#### VM_GC_Operation クラス関連のクラス (VM_GC_Operation, VM_GC_HeapInspection, VM_GenCollectForAllocation, VM_GenCollectFull, VM_GenCollectForPermanentAllocation, SvcGCMarker)

これらは, Garbage Collection 処理用の補助クラス.
より具体的に言うと, 実際の GC 処理を行うクラス (See: [here](no2480EWm.html) for details).


### クラス一覧(class list)

  * [VM_GC_Operation](#nooqxFK8q5)
  * [VM_GC_HeapInspection](#noXqr1RxRk)
  * [VM_GenCollectForAllocation](#non_3TpvyQ)
  * [VM_GenCollectFull](#noEq40gNR1)
  * [VM_GenCollectForPermanentAllocation](#noi2IoJO4D)
  * [SvcGCMarker](#noao-rw8R5)


---
## <a name="nooqxFK8q5" id="nooqxFK8q5">VM_GC_Operation</a>

### 概要(Summary)
Garbage Collection 処理を行う VM_Operation クラスの基底クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/vmGCOperations.hpp))
    class VM_GC_Operation: public VM_Operation {
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/vmGCOperations.hpp))
    //  VM_GC_Operation
    //   - implements methods common to all classes in the hierarchy:
    //     prevents multiple gc requests and manages lock on heap;
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classVM__GC__Operation.html) for details

---
## <a name="noXqr1RxRk" id="noXqr1RxRk">VM_GC_HeapInspection</a>

### 概要(Summary)
保守運用機能のためのクラス (関連する serviceability 機能からのみ使用される).

Java ヒープ内のオブジェクトに関する統計情報(どのクラスのインスタンスがどれだけ(何個および何バイト)存在しているか)を出力する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/vmGCOperations.hpp))
    class VM_GC_HeapInspection: public VM_GC_Operation {
```

### 使われ方(Usage)
以下の箇所で使用されている.

* CollectedHeap::pre_full_gc_dump()

  保守運用機能用の関数.
  GC 実行前に呼び出され, 関連するコマンドラインオプションの値に応じてヒープの情報を出力する.
  PrintClassHistogramBeforeFullGC オプションが指定されていると, VM_GC_HeapInspection が呼び出される.

* CollectedHeap::post_full_gc_dump()

  保守運用機能用の関数.
  GC 実行後に呼び出され, 関連するコマンドラインオプションの値に応じてヒープの情報を出力する.
  PrintClassHistogramAfterFullGC オプションが指定されていると, VM_GC_HeapInspection が呼び出される.

* signal_thread_entry

  (これはシグナルハンドラ用の関数. スレッドダンプ処理等を行う)

  PrintClassHistogram オプションが指定されていると,
  スレッドダンプ処理時(SIGBREAK の処理時)に VM_GC_HeapInspection が呼び出される

* heap_inspection()

  Attach API の "inspectheap" コマンドの処理を行う関数.

### 内部構造(Internal structure)
実際の処理は HeapInspection クラスに丸投げしているだけ (より正確には HeapInspection::heap_inspection() を呼び出しているだけ)
(See: HeapInspection).

なお, VM_GC_Operation のサブクラスなのに GC との関係が薄いが,
コンストラクタ引数で Full GC を実行してから調査するように要求されていた場合には,
まず Full GC を実行してから HeapInspection を呼び出す (このため, 一応 GC を行う処理パスも存在する).

#### 参考(for your information): VM_GC_HeapInspection::doit()
See: [here](no28916Gaj.html) for details



### 詳細(Details)
See: [here](../doxygen/classVM__GC__HeapInspection.html) for details

---
## <a name="non_3TpvyQ" id="non_3TpvyQ">VM_GenCollectForAllocation</a>

### 概要(Summary)
VM_GC_Operation クラスの具象サブクラスの1つ (See: [here](no2480EWm.html) for details).

このクラスは, GenCollectedHeap 用
(より正確には, GenCollectedHeap において New/Old 領域からの確保に失敗した場合用.
GC アルゴリズムとしては Serial, ParNew, Serial Old, CMS 等がここから呼び出される).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/vmGCOperations.hpp))
    class VM_GenCollectForAllocation: public VM_GC_Operation {
```

### 使われ方(Usage)
GenCollectorPolicy::mem_allocate_work() 内で(のみ)使用されている (See: [here](no28916sKh.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__GenCollectForAllocation.html) for details

---
## <a name="noEq40gNR1" id="noEq40gNR1">VM_GenCollectFull</a>

### 概要(Summary)
VM_GC_Operation クラスの具象サブクラスの1つ (See: [here](no2480EWm.html) for details).

このクラスは, GenCollectedHeap 用
(より正確には, GenCollectedHeap において java.lang.System.gc() 等が呼び出された場合用.
GC アルゴリズムとしては Serial Old がここから呼び出される).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/vmGCOperations.hpp))
    // VM operation to invoke a collection of the heap as a
    // GenCollectedHeap heap.
    class VM_GenCollectFull: public VM_GC_Operation {
```

### 使われ方(Usage)
GenCollectedHeap::collect_locked() 内で(のみ)使用されている (See: [here](no28916YTF.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__GenCollectFull.html) for details

---
## <a name="noi2IoJO4D" id="noi2IoJO4D">VM_GenCollectForPermanentAllocation</a>

### 概要(Summary)
VM_GC_Operation クラスの具象サブクラスの1つ (See: [here](no2480EWm.html) for details).

このクラスは, SharedHeap 用
(より正確には, GenCollectedHeap や G1CollectedHeap において Perm 領域からの確保に失敗した場合用.
GC アルゴリズムとしては Serial Old, CMS, G1GC 等がここから呼び出される).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/vmGCOperations.hpp))
    class VM_GenCollectForPermanentAllocation: public VM_GC_Operation {
```

### 使われ方(Usage)
PermGen::mem_allocate_in_gen() 内で(のみ)使用されている (See: [here](no28916-pc.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__GenCollectForPermanentAllocation.html) for details

---
## <a name="noao-rw8R5" id="noao-rw8R5">SvcGCMarker</a>

### 概要(Summary)
保守運用機能のためのクラス (DTrace 用および JVMTI 用のフック点を管理するためのクラス) 
(See: [here](no28916ldL.html), [here](no2935WqL.html) and [here](no2935j0R.html) for details).

DTrace や JVMTI のフック点の生成処理を簡単に行うための補助クラス(StackObjクラス).
ソースコード上のスコープに連動して自動的にフック点を生成する.
  

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/vmGCOperations.hpp))
    class SvcGCMarker : public StackObj {
```

### 使われ方(Usage)
コード中で SvcGCMarker 型の局所変数を宣言するだけ.

(以下の例のように GC 処理の開始前に局所変数が宣言される)

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/vmPSOperations.cpp))
    void VM_ParallelGCFailedAllocation::doit() {
      SvcGCMarker sgcm(SvcGCMarker::MINOR);
    ...
      _result = heap->failed_mem_allocate(_size, _is_tlab);
```

### 内部構造(Internal structure)
コンストラクタで VM_GC_Operation::notify_gc_begin() を, デストラクタで VM_GC_Operation::notify_gc_end() を呼び出す
(VM_GC_Operation::notify_gc_begin() や VM_GC_Operation::notify_gc_end() は DTrace のフック点.
 それぞれ gc__begin と gc__end に対応).

また, JvmtiGCMarker 型のフィールドを保持しているため JvmtiGCMarker のコンストラクタ／デストラクタも呼び出される
(この中で JVMTI のフック処理が行われる).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/vmGCOperations.hpp))
      SvcGCMarker(reason_type reason ) {
        VM_GC_Operation::notify_gc_begin(reason == FULL);
      }
    
      ~SvcGCMarker() {
        VM_GC_Operation::notify_gc_end();
      }
```

なおコンストラクタの方については,
「コンストラクタ引数として渡された reason が FULL かどうか」という情報も VM_GC_Operation::notify_gc_begin() に渡している.
この reason は以下の ３通りの値を取る enum 値.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/vmGCOperations.hpp))
      typedef enum { MINOR, FULL, OTHER } reason_type;
```

#### 参考(for your information): VM_GC_Operation::notify_gc_begin(), VM_GC_Operation::notify_gc_end()

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/vmGCOperations.cpp))
    // The same dtrace probe can't be inserted in two different files, so we
    // have to call it here, so it's only in one file.  Can't create new probes
    // for the other file anymore.   The dtrace probes have to remain stable.
    void VM_GC_Operation::notify_gc_begin(bool full) {
      HS_DTRACE_PROBE1(hotspot, gc__begin, full);
      HS_DTRACE_WORKAROUND_TAIL_CALL_BUG();
    }
    
    void VM_GC_Operation::notify_gc_end() {
      HS_DTRACE_PROBE(hotspot, gc__end);
      HS_DTRACE_WORKAROUND_TAIL_CALL_BUG();
    }
```




### 詳細(Details)
See: [here](../doxygen/classSvcGCMarker.html) for details

---

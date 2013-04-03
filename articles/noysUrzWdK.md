---
layout: default
title: SharedHeap クラス関連のクラス (SharedHeap, SharedHeap::StrongRootsScope, 及びそれらの補助クラス(AssertIsPermClosure, AssertNonScavengableClosure, AlwaysTrueClosure, SkipAdjustingSharedStrings))
---
[Top](../index.html)

#### SharedHeap クラス関連のクラス (SharedHeap, SharedHeap::StrongRootsScope, 及びそれらの補助クラス(AssertIsPermClosure, AssertNonScavengableClosure, AlwaysTrueClosure, SkipAdjustingSharedStrings))

これらは, Java ヒープ領域を管理するためのクラス (See: [here](no3718kvd.html) for details).

Java ヒープ領域を管理するクラスは使用する GC アルゴリズムによって異なるが,
これらのクラスは GC アルゴリズムが ParallelScavenge ではない場合に使用されるクラス (の基底クラス).
(See: GenCollectedHeap, G1CollectedHeap)



### クラス一覧(class list)

  * [SharedHeap](#noAWMjNakV)
  * [SharedHeap::StrongRootsScope](#noTnJVD8vy)
  * [AssertIsPermClosure](#no213OjMXB)
  * [AssertNonScavengableClosure](#no4TkBNWaK)
  * [AlwaysTrueClosure](#nolsiwJa_t)
  * [SkipAdjustingSharedStrings](#noXI6LHil-)


---
## <a name="noAWMjNakV" id="noAWMjNakV">SharedHeap</a>

### 概要(Summary)
CollectedHeap クラスのサブクラスの1つ (= Java ヒープ領域の管理を担当するクラス)

このクラスは, GC アルゴリズムが ParallelScavenge 以外の場合に使用される CollectedHeap クラス (の基底クラス)
(つまり GC アルゴリズムが Serial Old, CMS, または G1GC の場合用).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.hpp))
    // A "SharedHeap" is an implementation of a java heap for HotSpot.  This
    // is an abstract class: there may be many different kinds of heaps.  This
    // class defines the functions that a heap must implement, and contains
    // infrastructure common to all heaps.
```


```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.hpp))
    class SharedHeap : public CollectedHeap {
```




### 詳細(Details)
See: [here](../doxygen/classSharedHeap.html) for details

---
## <a name="noTnJVD8vy" id="noTnJVD8vy">SharedHeap::StrongRootsScope</a>

### 概要(Summary)
SharedHeap 用の MarkingCodeBlobClosure::MarkScope クラス (See: MarkingCodeBlobClosure::MarkScope).

MarkingCodeBlobClosure::MarkScope としての役割の他に,
SharedHeap::_strong_roots_parity の値をフリップさせるという役割を持っている.


```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.hpp))
      // Call these in sequential code around process_strong_roots.
      // strong_roots_prologue calls change_strong_roots_parity, if
      // parallel tasks are enabled.
      class StrongRootsScope : public MarkingCodeBlobClosure::MarkScope {
```

(なお SharedHeap::_strong_roots_parity は,
マルチスレッドで処理する GC の場合に
SharedHeap::process_strong_roots() 内の並列化しやすい処理 (threads の処理等) を簡単に並列化するための枠組み (備考参照))


```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.hpp))
      // Some collectors will perform "process_strong_roots" in parallel.
      // Such a call will involve claiming some fine-grained tasks, such as
      // scanning of threads.  To make this process simpler, we provide the
      // "strong_roots_parity()" method.  Collectors that start parallel tasks
      // whose threads invoke "process_strong_roots" must
      // call "change_strong_roots_parity" in sequential code starting such a
      // task.  (This also means that a parallel thread may only call
      // process_strong_roots once.)
      //
      // For calls to process_strong_roots by sequential code, the parity is
      // updated automatically.
      //
      // The idea is that objects representing fine-grained tasks, such as
      // threads, will contain a "parity" field.  A task will is claimed in the
      // current "process_strong_roots" call only if its parity field is the
      // same as the "strong_roots_parity"; task claiming is accomplished by
      // updating the parity field to the strong_roots_parity with a CAS.
      //
      // If the client meats this spec, then strong_roots_parity() will have
      // the following properties:
      //   a) to return a different value than was returned before the last
      //      call to change_strong_roots_parity, and
      //   c) to never return a distinguished value (zero) with which such
      //      task-claiming variables may be initialized, to indicate "never
      //      claimed".
```

### 内部構造(Internal structure)
コンストラクタで SharedHeap::change_strong_roots_parity() を呼び出して
SharedHeap::_strong_roots_parity の値を変更している
(なお, デストラクタでは特に何もしていない).


```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.cpp))
    SharedHeap::StrongRootsScope::StrongRootsScope(SharedHeap* outer, bool activate)
      : MarkScope(activate)
    {
      if (_active) {
        outer->change_strong_roots_parity();
      }
    }
    
    SharedHeap::StrongRootsScope::~StrongRootsScope() {
      // nothing particular
    }
```

### 備考(Notes)
コンストラクタから呼び出された SharedHeap::change_strong_roots_parity() では,
SharedHeap::_strong_roots_parity の値を 1~2 の間でフリップさせている
(正確には初期値が 0 なので 0 -> 1 -> 2 -> 1 -> 2 -> 1 -> ... だが,
最初の 0 は SharedHeap::strong_roots_parity() の返値としては見えないので実質 1~2 だけ).

そして, この SharedHeap::_strong_roots_parity の値は, 現状では以下の 2カ所で使われている.

  * Threads::possibly_parallel_oops_do()
  * SATBMarkQueueSet::par_iterate_closure_all_threads()

この2箇所での使われ方は以下の通り. これにより JavaThread という単位で GC 処理が並列化される.

  1. 予め GC 開始時に, SharedHeap::StrongRootsScope によって
     SharedHeap::change_strong_roots_parity() を呼び出しておく (これはシングルスレッドで行う).
  2. 以下の処理をマルチスレッドで行う.
     1. SharedHeap::_strong_roots_parity の値を取得する.
     2. 全ての JavaThread に対して Thread::claim_oops_do() を呼び出す.

        この中では, Atomic::cmpxchg() により, JavaThread::_oops_do_parity フィールドを
        1つ前の _strong_roots_parity の値から現在の _strong_roots_parity の値に書き換える処理が試みられる.

        書き換えが成功した GC スレッドがその JavaThread の処理を担当する.

#### 参考(for your information): _strong_roots_parity の初期値

```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.cpp))
    SharedHeap::SharedHeap(CollectorPolicy* policy_) :
    ...
      _strong_roots_parity(0),
```

#### 参考(for your information): SharedHeap::change_strong_roots_parity()
See: [here](no7882jSq.html) for details
#### 参考(for your information): Threads::possibly_parallel_oops_do()
See: [here](no7882WWM.html) for details
#### 参考(for your information): SATBMarkQueueSet::par_iterate_closure_all_threads()
See: [here](no7882JMG.html) for details
#### 参考(for your information): Thread::claim_oops_do()
See: [here](no78828BA.html) for details
#### 参考(for your information): Thread::claim_oops_do_par_case()
See: [here](no7882K4w.html) for details



### 詳細(Details)
See: [here](../doxygen/classSharedHeap_1_1StrongRootsScope.html) for details

---
## <a name="no213OjMXB" id="no213OjMXB">AssertIsPermClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

処理対象の oop* が Perm 領域内にあることを検証する.


```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.cpp))
    class AssertIsPermClosure: public OopClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
assert_is_perm_closure という大域変数に(のみ)格納されている.


```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.cpp))
    static AssertIsPermClosure assert_is_perm_closure;
```

#### 使用箇所(where its instances are used)
SharedHeap::process_strong_roots() 内で, 
StringTable 内の文字列が Perm 領域内に入っていることを確かめるために使われている.

(なおこの処理は, NOT_PRODUCT 時で, かつ 
develop オプションである JavaObjectsInPerm が指定されている場合でないと実行されない)


```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.cpp))
    void SharedHeap::process_strong_roots(bool activate_scope,
                                          bool collecting_perm_gen,
                                          ScanningOption so,
                                          OopClosure* roots,
                                          CodeBlobClosure* code_roots,
                                          OopsInGenClosure* perm_blk) {
    ...
        if (JavaObjectsInPerm) {
          // Verify the string table contents are in the perm gen
          NOT_PRODUCT(StringTable::oops_do(&assert_is_perm_closure));
```




### 詳細(Details)
See: [here](../doxygen/classAssertIsPermClosure.html) for details

---
## <a name="no4TkBNWaK" id="no4TkBNWaK">AssertNonScavengableClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).

処理対象の oop* が Minor GC の処理対象範囲内にないことを検証する.


```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.cpp))
    #ifdef ASSERT
    class AssertNonScavengableClosure: public OopClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
assert_is_non_scavengable_closure という大域変数に(のみ)格納されている.


```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.cpp))
    static AssertNonScavengableClosure assert_is_non_scavengable_closure;
```

#### 使用箇所(where its instances are used)
SharedHeap::process_strong_roots() 内で, 
CodeCache 内の CodeBlob には New 領域内を指すポインタを持っているものは存在しない, 
ということを確かめるために使われている.

(なおこの処理は, DEBUG_ONLY 時にしか実行されない)


```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.cpp))
    void SharedHeap::process_strong_roots(bool activate_scope,
                                          bool collecting_perm_gen,
                                          ScanningOption so,
                                          OopClosure* roots,
                                          CodeBlobClosure* code_roots,
                                          OopsInGenClosure* perm_blk) {
    ...
        // Verify that the code cache contents are not subject to
        // movement by a scavenging collection.
        DEBUG_ONLY(CodeBlobToOopClosure assert_code_is_non_scavengable(&assert_is_non_scavengable_closure, /*do_marking=*/ false));
        DEBUG_ONLY(CodeCache::asserted_non_scavengable_nmethods_do(&assert_code_is_non_scavengable));
```




### 詳細(Details)
See: [here](../doxygen/classAssertNonScavengableClosure.html) for details

---
## <a name="nolsiwJa_t" id="nolsiwJa_t">AlwaysTrueClosure</a>

### 概要(Summary)
SharedHeap::process_weak_roots() 内で使用される補助クラス
(つまり, JNI の Weak Global Handle を辿る処理で使用される Closure クラス).

名前の通り, どんな場合でも常に true を返す BoolObjectClosure.


```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.cpp))
    class AlwaysTrueClosure: public BoolObjectClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
always_true という大域変数に(のみ)格納されている.


```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.cpp))
    static AlwaysTrueClosure always_true;
```

#### 使用箇所(where its instances are used)
SharedHeap::process_weak_roots() 内で(のみ)使用されている.

(なお, phase1 の中で ReferenceProcessor::process_discovered_references() が呼び出された段階で, 
 死んでいる Weak Global Reference については NULL になっている.
 このため, AlwaysTrueClosure のような常に true を返すだけの Closure でも, 
 生きている Weak Global Reference だけを全て辿ることができる.)


```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.cpp))
    void SharedHeap::process_weak_roots(OopClosure* root_closure,
                                        CodeBlobClosure* code_roots,
                                        OopClosure* non_root_closure) {
      // Global (weak) JNI handles
      JNIHandles::weak_oops_do(&always_true, root_closure);
```




### 詳細(Details)
See: [here](../doxygen/classAlwaysTrueClosure.html) for details

---
## <a name="noXI6LHil-" id="noXI6LHil-">SkipAdjustingSharedStrings</a>

### 概要(Summary)
SharedHeap::process_weak_roots() 内で使用される補助クラス
(つまり, JNI の Weak Global Handle を辿る処理で使用される Closure クラス).


```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.cpp))
    class SkipAdjustingSharedStrings: public OopClosure {
```

### 使われ方(Usage)

```
    ((cite: hotspot/src/share/vm/memory/sharedHeap.cpp))
    void SharedHeap::process_weak_roots(OopClosure* root_closure,
                                        CodeBlobClosure* code_roots,
                                        OopClosure* non_root_closure) {
    ...
      if (UseSharedSpaces && !DumpSharedSpaces) {
        SkipAdjustingSharedStrings skip_closure(root_closure);
        StringTable::oops_do(&skip_closure);
```




### 詳細(Details)
See: [here](../doxygen/classSkipAdjustingSharedStrings.html) for details

---

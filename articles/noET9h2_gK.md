---
layout: default
title: SparsePRT クラス関連のクラス (SparsePRTEntry, RSHashTable, RSHashTableIter, SparsePRT, SparsePRTIter, SparsePRTCleanupTask)
---
[Top](../index.html)

#### SparsePRT クラス関連のクラス (SparsePRTEntry, RSHashTable, RSHashTableIter, SparsePRT, SparsePRTIter, SparsePRTCleanupTask)

これらは, G1CollectedHeap 使用時における Garbage Collection 処理用の補助クラス.
より具体的に言うと, Remembered Set の一種 (See: [here](no3718kvd.html) for details).

### 概要(Summary)
SparsePRT は OtherRegionsTable クラス内で使用される補助クラス (なお PRT は PerRegionTables の略(だと思われる)).
細かい粒度(Card 単位)での参照情報を記録するために使用される.

中身は, HeapRegion の index から Card 集合への写像, といった感じ.
HeapRegion の index を渡すと「その HeapRegion 内のどの card から指されているか」という情報を返してくれる.

なおコメントによると, 
「parallel にアクセスされた際にのみ拡張が起こり, 削除処理はシングルスレッドでのみ実行するので,
reads/iterations は lock を取らずに実行してよい」とのこと
(ただし, 挿入時に拡張が起きた場合は古いものを delete 対象として予約するだけに止めておかないといけない. 実際に消しちゃったらダメ).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/sparsePRT.hpp))
    // Sparse remembered set for a heap region (the "owning" region).  Maps
    // indices of other regions to short sequences of cards in the other region
    // that might contain pointers into the owner region.
    
    // These tables only expand while they are accessed in parallel --
    // deletions may be done in single-threaded code.  This allows us to allow
    // unsynchronized reads/iterations, as long as expansions caused by
    // insertions only enqueue old versions for deletions, but do not delete
    // old versions synchronously.
```

なお, これらのクラスは G1HRRSUseSparseTable オプションが変更されていない場合にのみ使用される
(といっても develop オプションなので通常時には変更できないが).

(<= ちなみに, OtherRegionsTable::contains_reference_locked() 内で使用する際には
 G1HRRSUseSparseTable オプションの値を参照していないが, 
 _coarse_map.at() も find_region_table() も試した後で参照しているから, 
 G1HRRSUseSparseTable オプションが指定されてない場合には意味は無いんだろう)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1_globals.hpp))
      develop(bool, G1HRRSUseSparseTable, true,                                 \
              "When true, use sparse table to save space.")                     \
```

なお, SparsePRT が一杯になると, 代わりに PosParPRT が使用されるようになる (See: OtherRegionsTable).
その際には対応する SparsePRT から PosParPRT にデータが移され, SparsePRT は開放される 
(See: OtherRegionsTable::add_reference()).



### クラス一覧(class list)

  * [SparsePRT](#noFcOztUVe)
  * [RSHashTable](#noWJepoNb0)
  * [SparsePRTEntry](#no_zouT7TY)
  * [RSHashTableIter](#noE0Bb3jDR)
  * [SparsePRTIter](#noZ-KSqjhv)
  * [SparsePRTCleanupTask](#nobtUyRHrx)


---
## <a name="noFcOztUVe" id="noFcOztUVe">SparsePRT</a>

### 概要(Summary)
OtherRegionsTable クラス内で使用される補助クラス.

細かい粒度(Card 単位)での参照情報を記録するためのクラス.
「各 HeapRegion 内のどの card から指されているか」という情報を格納している.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/sparsePRT.hpp))
    class SparsePRT VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 OtherRegionsTable オブジェクトの _sparse_table フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(OtherRegionsTable クラスの _sparse_table フィールドは, ポインタ型ではなく実体なので,
 OtherRegionsTable オブジェクトの生成時に一緒に生成される)

### 内部構造(Internal structure)
実際の Remembered Set 情報は RSHashTable オブジェクト (の中の SparsePRTEntry オブジェクト) に格納されている.

SparsePRT オブジェクトでは 2種類の RSHashTable オブジェクトを保持している.

* _cur は, GC 開始時の状態を表す. GC 中に参照されるのはこちら.
* 変更は全て _next に対して行われる. GC 後に _next の内容が _cur に移される.

### 備考(Notes)
拡張(expand)された SparsePRT オブジェクトは, _head_expanded_list 大域変数につながれている (See: SparsePRT::expand()).

```
SparsePRT::expand()
-> SparsePRT::add_to_expanded_list()
```

(ただし, expand しても _next が realloc されるだけでその時点では増えない. 
GC 時に _cur を _next に置き換える処理が行われる (See: G1RemSet::cleanupHRRS()))

_head_expanded_list 大域変数につながれた SparsePRT オブジェクトは, 
SparsePRT::get_from_expanded_list() で取得されて処理される.

```
G1CollectedHeap::do_collection()
-> G1RemSet::cleanupHRRS()
   -> HeapRegionRemSet::cleanup()
      -> SparsePRT::cleanup_all()
         -> SparsePRT::get_from_expanded_list()
         -> SparsePRT::cleanup()

G1CollectedHeap::evacuate_collection_set()
-> G1RemSet::prepare_for_oops_into_collection_set_do()
   -> G1RemSet::cleanupHRRS()
      -> (同上)

G1RemSet::prepare_for_verify()
-> G1RemSet::cleanupHRRS()
   -> (同上)
```


なお, _head_expanded_list 大域変数のリストは Concurrent Marking 処理が起こると
(Concurrent Marking で回収された SparsePRT を含まない状態へと) 再構築される.
この処理は Concurrent Marking の Cleanup 処理で行われている.

* SparsePRT::reset_for_cleanup_tasks() で, リストが空にリセットされる.
* Concurrent Marking 中では, 再構築のため, 拡張(expand)された SparsePRT オブジェクトが検出される.
  (これは G1NoteEndOfConcMarkClosure 内で呼び出される G1CollectedHeap::free_region_if_empty() で行われている)
* 検出された SparsePRT オブジェクトは, ConcurrentMarkThread が呼ぶ HeapRegionRemSet::finish_cleanup_task() で 
  _head_expanded_list 大域変数につなぎ直される.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/sparsePRT.hpp))
      // The purpose of these three methods is to help the GC workers
      // during the cleanup pause to recreate the expanded list, purging
      // any tables from it that belong to regions that are freed during
      // cleanup (if we don't purge those tables, there is a race that
      // causes various crashes; see CR 7014261).
      //
      // We chose to recreate the expanded list, instead of purging
      // entries from it by iterating over it, to avoid this serial phase
      // at the end of the cleanup pause.
      //
      // The three methods below work as follows:
      // * reset_for_cleanup_tasks() : Nulls the expanded list head at the
      //   start of the cleanup pause.
      // * do_cleanup_work() : Called by the cleanup workers for every
      //   region that is not free / is being freed by the cleanup
      //   pause. It creates a list of expanded tables whose head / tail
      //   are on the thread-local SparsePRTCleanupTask object.
      // * finish_cleanup_task() : Called by the cleanup workers after
      //   they complete their cleanup task. It adds the local list into
      //   the global expanded list. It assumes that the
      //   ParGCRareEvent_lock is being held to ensure MT-safety.
      static void reset_for_cleanup_tasks();
      void do_cleanup_work(SparsePRTCleanupTask* sprt_cleanup_task);
      static void finish_cleanup_task(SparsePRTCleanupTask* sprt_cleanup_task);
```




### 詳細(Details)
See: [here](../doxygen/classSparsePRT.html) for details

---
## <a name="noWJepoNb0" id="noWJepoNb0">RSHashTable</a>

### 概要(Summary)
SparsePRT クラス内で使用される補助クラス.

SparsePRTEntry オブジェクトを束ねておくためのコンテナクラス
(各 SparsePRTEntry オブジェクトが 1つの HeapRegion に対応し, 
 RSHashTable オブジェクトが HeapRegion の集合に対応する).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/sparsePRT.hpp))
    class RSHashTable : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 SparsePRT オブジェクトの _cur フィールド
* 各 SparsePRT オブジェクトの _next フィールド


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/sparsePRT.hpp))
      //  Iterations are done on the _cur hash table, since they only need to
      //  see entries visible at the start of a collection pause.
      //  All other operations are done using the _next hash table.
      RSHashTable* _cur;
      RSHashTable* _next;
```

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* SparsePRT::SparsePRT() 
* SparsePRT::clear()
* SparsePRT::expand()

### 内部構造(Internal structure)
内部に含まれる SparsePRTEntry オブジェクトの個数は動的に増加する
(最初はコンストラクタ引数で指定された個数だが, 
 SparsePRT::expand() が呼び出されると 2倍に増加する).




### 詳細(Details)
See: [here](../doxygen/classRSHashTable.html) for details

---
## <a name="no_zouT7TY" id="no_zouT7TY">SparsePRTEntry</a>

### 概要(Summary)
SparsePRT クラス内で使用される補助クラス.

実際の SparsePRT の中身を表すクラス.
この中に「どの card から指されているか」という情報が格納されている.

なお, 1つの SparsePRTEntry は 1つの HeapRegion からの参照のみを記録している
(= 「その HeapRegion 内のどの card から指されているか」という情報を記録している).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/sparsePRT.hpp))
    class SparsePRTEntry: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 RSHashTable オブジェクトの _entries フィールドに(のみ)格納されている

(正確には, このフィールドは SparsePRTEntry の配列を格納するフィールド.
この中に, その RSHashTable 内で使用される全ての SparsePRTEntry オブジェクトが格納されている).

#### 生成箇所(where its instances are created)
配列用のメモリ領域は RSHashTable::RSHashTable() 内で(のみ)確保されている. 

そのメモリ領域中に個別の SparsePRTEntry オブジェクトを書き込む作業は, SparsePRTEntry::add_card() 内で(のみ)行われている.

### 内部構造(Internal structure)
内部には, SparsePRTEntry::cards_num() 個数分の CardIdx_t 配列を持つ
(宣言では 1 個だが実体は違う).

ここに, 「その HeapRegion 内のどの card から指されているか」という情報を記録している.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/sparsePRT.hpp))
      CardIdx_t   _cards[1];
```

なお, SparsePRTEntry::cards_num() は以下の値のどちらか大きい方 (See: SparsePRTEntry::cards_num()).

* G1RSetSparseRegionEntries オプションの値 (を UnrollFactor の倍数にしたもの)
  
  なお, ユーザーが指定しなかった場合は ergonomics 的に決定される
  (See: HeapRegionRemSet::setup_remset_size()).

* UnrollFactor (= 4)
  
  (これは, SparsePRTEntry 用の処理が手動でループアンローリングされているのでその倍数に合わせるための定数.
   なお, 現状では 4 以外にする場合は該当箇所のソースコードも修正しないといけない)


(なお, 配列中で未使用の箇所は SparsePRTEntry::NullEntry になっている (See: SparsePRTEntry::init()))




### 詳細(Details)
See: [here](../doxygen/classSparsePRTEntry.html) for details

---
## <a name="noE0Bb3jDR" id="noE0Bb3jDR">RSHashTableIter</a>

### 概要(Summary)
SparsePRT 内の要素を辿るためのイテレータクラスの基底クラス
(より正確に言うと RSHashTable 内の要素をたどるためのイテレータクラスの基底クラス).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/sparsePRT.hpp))
    // ValueObj because will be embedded in HRRS iterator.
    class RSHashTableIter VALUE_OBJ_CLASS_SPEC {
```




### 詳細(Details)
See: [here](../doxygen/classRSHashTableIter.html) for details

---
## <a name="noZ-KSqjhv" id="noZ-KSqjhv">SparsePRTIter</a>

### 概要(Summary)
HeapRegionRemSetIterator クラス内で使用される補助クラス.

RSHashTableIter クラスの具象サブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/sparsePRT.hpp))
    class SparsePRTIter: public RSHashTableIter {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. SparsePRTIter::init() で初期化する
2. SparsePRTIter::has_next() で次の card_index を取得する
   (なお, card_index は返値ではなく参照渡しした引数にセットされて返される.
    返値は要素がまだ残っているかどうかを示すために使われており, 返値が false になったら終了)

#### インスタンスの格納場所(where its instances are stored)
各 HeapRegionRemSetIterator オブジェクトの _sparse_iter フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(HeapRegionRemSetIterator クラスの _sparse_iter フィールドは, ポインタ型ではなく実体なので,
 HeapRegionRemSetIterator オブジェクトの生成時に一緒に生成される)

#### 使用箇所(where its instances are used)
HeapRegionRemSetIterator::has_next() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classSparsePRTIter.html) for details

---
## <a name="nobtUyRHrx" id="nobtUyRHrx">SparsePRTCleanupTask</a>

### 概要(Summary)
SparsePRT の処理用の補助クラス(の基底クラス).

拡張(expand)された SparsePRT オブジェクトを元に戻す処理で使用されるクラス.
なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス
(See: HRRSCleanupTask).

Concurrent Marking 処理の Cleanup 処理中に 
拡張(expand)された SparsePRT オブジェクトがこのオブジェクト内に収集される.
なお, このオブジェクトは Thread local に使用される.
Cleanup 処理の終了時に全スレッド分の内容が 1つにまとめられ, 次回の GC 時に処理される
(See: SparsePRT::_head_expanded_list, HeapRegionRemSet::finish_cleanup_task()).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/sparsePRT.hpp))
    // This allows each worker during a cleanup pause to create a
    // thread-local list of sparse tables that have been expanded and need
    // to be processed at the beginning of the next GC pause. This lists
    // are concatenated into the single expanded list at the end of the
    // cleanup pause.
    class SparsePRTCleanupTask VALUE_OBJ_CLASS_SPEC {
```




### 詳細(Details)
See: [here](../doxygen/classSparsePRTCleanupTask.html) for details

---

---
layout: default
title: HeapRegionRemSet クラス関連のクラス (HRRSCleanupTask, OtherRegionsTable, HeapRegionRemSet, HeapRegionRemSetIterator, CardClosure, 及びそれらの補助クラス(PerRegionTable, PosParPRT))
---
[Top](../index.html)

#### HeapRegionRemSet クラス関連のクラス (HRRSCleanupTask, OtherRegionsTable, HeapRegionRemSet, HeapRegionRemSetIterator, CardClosure, 及びそれらの補助クラス(PerRegionTable, PosParPRT))

これらは, G1CollectedHeap 使用時における Garbage Collection 処理用の補助クラス.
より具体的に言うと, Remembered Set の一種 (See: [here](no3718kvd.html) for details).


### クラス一覧(class list)

  * [HeapRegionRemSet](#no2XhC65n1)
  * [OtherRegionsTable](#noHWaeBHHM)
  * [PerRegionTable](#no-2uaGR7s)
  * [PosParPRT](#noKVSOXDNB)
  * [HeapRegionRemSetIterator](#noUgZ9JF7p)
  * [HRRSCleanupTask](#no8mjpESF-)
  * [CardClosure](#nogp8N8lzs)


---
## <a name="no2XhC65n1" id="no2XhC65n1">HeapRegionRemSet</a>

### 概要(Summary)
G1GC 使用時における Remembered Set 機能を提供するクラス.

HeapRegion 毎に1つ存在しており, 
「その HeapRegion を指すポインタ (を含んでいる Card/HeapRegion) の集合」を記録している.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.hpp))
    // Remembered set for a heap region.  Represent a set of "cards" that
    // contain pointers into the owner heap region.  Cards are defined somewhat
    // abstractly, in terms of what the "BlockOffsetTable" in use can parse.
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.hpp))
    class HeapRegionRemSet : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 HeapRegion オブジェクトの _rem_set フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
HeapRegion::HeapRegion() 内で(のみ)生成されている.

### 内部構造(Internal structure)
実際の Remembered Set 機能は, OtherRegionsTable によって実現されている (See: OtherRegionsTable).




### 詳細(Details)
See: [here](../doxygen/classHeapRegionRemSet.html) for details

---
## <a name="noHWaeBHHM" id="noHWaeBHHM">OtherRegionsTable</a>

### 概要(Summary)
HeapRegionRemSet クラス内で使用される補助クラス.
実際の Remembered Set としての機能はこのクラスが提供している.

指定の HeapRegion を指している Card/HeapRegion を記録しておくためのクラス.
内部的には 3種類のデータ構造を使い分けている
(まず _sparse_table を使用し, 一杯になったら _fine_grain_regions を使用し, 
 それも一杯になったら _coarse_map が使用される).

  * _coarse_map フィールドは, 粗い粒度(HeapRegion 単位)での情報を蓄えておくフィールド.
    内部的には 1bit が 1 HeapRegion に対応するビットマップ
    (その HeapRegion から指されていればビットが1になる).

  * _fine_grain_regions フィールドは, 細かい粒度(Card 単位)での情報を蓄えておくフィールド.
    どの Card から指されているかを記録している.
    内部的には PerRegionTables (PRT) を要素とする open hash.

  * (コメントには書かれていないが, G1HRRSUseSparseTable オプションが指定されていれば, 
    細かい粒度(Card 単位)での情報を蓄えておくフィールドとして _sparse_table も使われる.
    このフィールドは, 内部的には SparsePRT)

(なおコメントによると, 
 「読むときには lock は取らなくていいが, 変更時には lock を取ることにしている
 (というわけで, hash に見つからなかった場合はロックを取ることになる).
 並行して削除処理が行われている要素を見つけてしまうとまずい気もするが,
 本当にメモリを解放するのは safepoint でしかやらないし, 
 読むのも要素を追加するときだけなので coarse に追い出された要素を見てしまっても特に問題は無い」
 とのこと.)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.hpp))
    // The "_coarse_map" is a bitmap with one bit for each region, where set
    // bits indicate that the corresponding region may contain some pointer
    // into the owning region.
    
    // The "_fine_grain_entries" array is an open hash table of PerRegionTables
    // (PRTs), indicating regions for which we're keeping the RS as a set of
    // cards.  The strategy is to cap the size of the fine-grain table,
    // deleting an entry and setting the corresponding coarse-grained bit when
    // we would overflow this cap.
    
    // We use a mixture of locking and lock-free techniques here.  We allow
    // threads to locate PRTs without locking, but threads attempting to alter
    // a bucket list obtain a lock.  This means that any failing attempt to
    // find a PRT must be retried with the lock.  It might seem dangerous that
    // a read can find a PRT that is concurrently deleted.  This is all right,
    // because:
    //
    //   1) We only actually free PRT's at safe points (though we reuse them at
    //      other times).
    //   2) We find PRT's in an attempt to add entries.  If a PRT is deleted,
    //      it's _coarse_map bit is set, so the that we were attempting to add
    //      is represented.  If a deleted PRT is re-used, a thread adding a bit,
    //      thinking the PRT is for a different region, does no harm.
    
    class OtherRegionsTable VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 HeapRegionRemSet オブジェクトの _other_regions フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(HeapRegionRemSet クラスの _other_regions フィールドは, ポインタ型ではなく実体なので,
 HeapRegionRemSet オブジェクトの生成時に一緒に生成される)

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

* G1CollectedHeap* _g1h


* Mutex            _m
  
  

* HeapRegion*      _hr
  
  対応する HeapRegion

* BitMap      _coarse_map
  
  粗い粒度(HeapRegion 単位)での情報を蓄えておくフィールド.
  1bit が 1 HeapRegion に対応するビットマップ.
  その HeapRegion から指されていればビットが1になる.

* size_t      _n_coarse_entries
  
  _coarse_map 内に入っている要素数

* PosParPRT** _fine_grain_regions
  
  細かい粒度(Card 単位)での情報を蓄えておくフィールド (See: PosParPRT)

* size_t      _n_fine_entries
  
  _fine_grain_regions 内に入っている要素数

* size_t        _fine_eviction_start
  
  SAMPLE_FOR_EVICTION 処理用のフィールド (See: OtherRegionsTable::delete_region_table())

* static size_t _fine_eviction_stride
  
  SAMPLE_FOR_EVICTION 処理用のフィールド (See: OtherRegionsTable::delete_region_table())

* static size_t _fine_eviction_sample_size

  SAMPLE_FOR_EVICTION 処理用のフィールド (See: OtherRegionsTable::delete_region_table())

* SparsePRT   _sparse_table
  
  細かい粒度(Card 単位)での情報を蓄えておくフィールド (See: SparsePRT).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.hpp))
      G1CollectedHeap* _g1h;
      Mutex            _m;
      HeapRegion*      _hr;
    
      // These are protected by "_m".
      BitMap      _coarse_map;
      size_t      _n_coarse_entries;
      static jint _n_coarsenings;
    
      PosParPRT** _fine_grain_regions;
      size_t      _n_fine_entries;
    
    #define SAMPLE_FOR_EVICTION 1
    #if SAMPLE_FOR_EVICTION
      size_t        _fine_eviction_start;
      static size_t _fine_eviction_stride;
      static size_t _fine_eviction_sample_size;
    #endif
    
      SparsePRT   _sparse_table;
```




### 詳細(Details)
See: [here](../doxygen/classOtherRegionsTable.html) for details

---
## <a name="no-2uaGR7s" id="no-2uaGR7s">PerRegionTable</a>

### 概要(Summary)
PosParPRT クラス用の補助クラス.

指定の HeapRegion が「どの Card から指されているか」という情報を記録しておくためのクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.cpp))
    class PerRegionTable: public CHeapObj {
```

なお, 1つの PerRegionTable オブジェクトは 1つの HeapRegion からの参照のみを記録している
(つまり「その HeapRegion 内のどの card から指されているか」という情報を記録している).
ある HeapRegion を指している全ての Card の情報は, 複数の PerRegionTable オブジェクトを用いることで表現される.

### 使われ方(Usage)
#### 使用方法の概要(how to use)
PerRegionTable クラスのサブクラスとして PosParPRT クラスがあり, 
通常時は PosParPRT だけが使用されている.

ただし GC 処理中には, マルチスレッドでの処理を効率化するため, 
PosParPRT 内で追加の PerRegionTable オブジェクトが確保される
(各 GC Thread が自分用の PerRegionTable オブジェクトを持って処理を行う)
(このため, これと区別するために PosParPRT 自身を "base table" と呼んだりしている模様).

#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 PosParPRT オブジェクトの _par_tables フィールド

  (正確には, このフィールドは PerRegionTable の配列を格納するフィールド.
   この中に, その PosParPRT 内で使用される全ての PerRegionTable オブジェクトが格納されている.
   配列長は HeapRegionRemSet::num_par_rem_sets())
  
  (なお, このフィールドにオブジェクトが格納されているのは GC 処理中のみ. 
   GC 処理後には HeapRegionRemSet::par_cleanup() で解放されるため通常時は空)

* PerRegionTable クラスの _free_list フィールド (static フィールド)
  
  (正確には, このフィールドは PerRegionTable の線形リストを格納するフィールド(フリーリスト).
  PerRegionTable オブジェクトは _next_free フィールドで次の PerRegionTable オブジェクトを指せる構造になっている. 
  このフィールドの線形リストに全ての未使用な PerRegionTable オブジェクトがつながれている.)

(なお, この他に PosParPRT クラスに _par_table_fl フィールド (static フィールド) というフィールドがあるが, 
 これはどこからも使用されていない... 何のためのフィールド?? #TODO)  

#### 生成箇所(where its instances are created)
PerRegionTable::alloc() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
(略) (See: [here](no2935YzN.html) for details)
-> OtherRegionsTable::add_reference()
   -> PosParPRT::par_expand()
      -> PerRegionTable::alloc()
```

### 内部構造(Internal structure)
内部的には BitMap クラスを用いて情報を記録している
(「担当する HeapRegion 内のどの card から指されているか」という情報がビットマップとして記録されている).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.cpp))
      BitMap          _bm;
```




### 詳細(Details)
See: [here](../doxygen/classPerRegionTable.html) for details

---
## <a name="noKVSOXDNB" id="noKVSOXDNB">PosParPRT</a>

### 概要(Summary)
OtherRegionsTable クラス内で使用される補助クラス
(なお PRT は PerRegionTables の略(だと思われる)).

指定の HeapRegion が「どの Card から指されているか」という情報を記録しておくためのクラス.
(なお, PerRegionTable のサブクラスなので, 基本的に PerRegionTable クラスと同じ.
 1つの PerRegionTable は 1つの HeapRegion からの参照のみを記録しており, 
 ある HeapRegion を指している全ての Card 情報は複数の PosParPRT オブジェクトを用いることで表現される).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.cpp))
    class PosParPRT: public PerRegionTable {
```

なお PosParPRTPtr という型も使われるが, これは PosParPRT* の別名.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.cpp))
      typedef PosParPRT* PosParPRTPtr;
```


### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 OtherRegionsTable オブジェクトの _fine_grain_regions フィールド
  
  (正確には, このフィールドは PosParPRT の配列を格納するフィールド.
  この中に, その OtherRegionsTable 内で使用される全ての PosParPRT オブジェクトが格納されている)

* PosParPRT クラスの _free_list フィールド (static フィールド)

  (正確には, このフィールドは PosParPRT の線形リストを格納するフィールド(フリーリスト).
  PosParPRT オブジェクトは _next フィールドで次の PosParPRT オブジェクトを指せる構造になっている. 
  このフィールドの線形リストに全ての未使用な PosParPRT オブジェクトがつながれている.)

* PosParPRT クラスの _par_expanded_list フィールド (static フィールド)

  (正確には, このフィールドは PosParPRT の線形リストを格納するフィールド.
  PosParPRT オブジェクトは _next フィールドで次の PosParPRT オブジェクトを指せる構造になっている. 
  このフィールドの線形リストに, GC 中に PosParPRT::par_expand() で拡張された全ての PosParPRT オブジェクトがつながれている.
  この情報は, GC 処理後に HeapRegionRemSet::par_cleanup() で解放するために使用される)

#### 生成箇所(where its instances are created)
PosParPRT::alloc() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
(略) (See: [here](no2935YzN.html) and [here](no3420WIS.html) for details)
-> OtherRegionsTable::add_reference()
   -> PosParPRT::alloc()
```

(なお, OtherRegionsTable::_fine_grain_regions フィールドの配列用のメモリ領域は, 
 OtherRegionsTable::OtherRegionsTable() 内で(のみ)確保されている)




### 詳細(Details)
See: [here](../doxygen/classPosParPRT.html) for details

---
## <a name="noUgZ9JF7p" id="noUgZ9JF7p">HeapRegionRemSetIterator</a>

### 概要(Summary)
HeapRegionRemSet オブジェクト内の card 情報をたどるためのイテレータクラス
(なお, このクラスはイテレータなのに CHeapObj).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.hpp))
    class HeapRegionRemSetIterator : public CHeapObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. HeapRegionRemSet::init_iterator() で初期化する
2. HeapRegionRemSetIterator.has_next() で次の card_index を取得する
   (なお, card_index は返値ではなく参照渡しした引数にセットされて返される.
    返値は要素がまだ残っているかどうかを示すために使われており, 返値が false になったら終了)

#### インスタンスの格納場所(where its instances are stored)
各 G1CollectedHeap オブジェクトの _rem_set_iterator フィールドに(のみ)格納されている

(正確には, このフィールドは HeapRegionRemSetIterator の配列を格納するフィールド.
この中に, 使用される全ての HeapRegionRemSetIterator オブジェクトが格納されている).

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている
(ただし, 後ろの 2つは局所変数としての生成なので一時的なオブジェクト. またデバッグ時(開発時)にしか呼び出されない).

* G1CollectedHeap::G1CollectedHeap() 内
  
  G1CollectedHeap::_rem_set_iterator フィールドに格納する HeapRegionRemSetIterator オブジェクトを生成する.
  配列用のメモリ空間の確保もここで行う.

* HeapRegionRemSet::print() 内 (局所変数として生成)
  
  ただし, この関数はデバッグ用(開発時用). #ifndef PRODUCT 時でないと定義されない

* HeapRegionRemSet::test() 内 (局所変数として生成)
  
  ただし, この関数はデバッグ用(開発時用). #ifndef PRODUCT 時でないと定義されない

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* ScanRSClosure::doHeapRegion() 内
  
  各 GCThread が, G1CollectedHeap::_rem_set_iterator フィールドから, 
  自分の worker id に対応する HeapRegionRemSetIterator オブジェクトを取得して使用.

* HeapRegionRemSet::print() 内  (ただし, この関数はデバッグ用(開発時用). #ifndef PRODUCT 時でないと定義されない)

* HeapRegionRemSet::test() 内  (ただし, この関数はデバッグ用(開発時用). #ifndef PRODUCT 時でないと定義されない)




### 詳細(Details)
See: [here](../doxygen/classHeapRegionRemSetIterator.html) for details

---
## <a name="no8mjpESF-" id="no8mjpESF-">HRRSCleanupTask</a>

### 概要(Summary)
SparsePRTCleanupTask クラスの具象サブクラス (コメントによると, SparsePRTCleanupTask のラッパーのようなクラス, とのこと).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.hpp))
    // Essentially a wrapper around SparsePRTCleanupTask. See
    // sparsePRT.hpp for more details.
    class HRRSCleanupTask : public SparsePRTCleanupTask {
```

### 使われ方(Usage)
G1ParNoteEndTask::work() 内の局所変数として(のみ)生成されている.

### 内部構造(Internal structure)
何も定義されていない (なので中身はスーパークラスである SparsePRTCleanupTask と全く同じ) 
(なんでこんなクラスが作られた?? #TODO).




### 詳細(Details)
See: [here](../doxygen/classHRRSCleanupTask.html) for details

---
## <a name="nogp8N8lzs" id="nogp8N8lzs">CardClosure</a>

### 概要(Summary)
?? (使われていないクラス. #if 0 で消されていて詳細不明)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.hpp))
    #if 0
    class CardClosure: public Closure {
```




### 詳細(Details)
See: [here](../doxygen/classCardClosure.html) for details

---

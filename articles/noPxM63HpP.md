---
layout: default
title: HeapRegionSet クラスのサブクラス群 (FreeRegionList, MasterFreeRegionList, SecondaryFreeRegionList, HumongousRegionSet, MasterHumongousRegionSet)
---
[Top](../index.html)

#### HeapRegionSet クラスのサブクラス群 (FreeRegionList, MasterFreeRegionList, SecondaryFreeRegionList, HumongousRegionSet, MasterHumongousRegionSet)

これらは, G1GC で使用するメモリ領域を管理するためのクラス.
より具体的に言うと, 未使用の HeapRegion や Humongous オブジェクトの先頭となっている HeapRegin を管理するためのクラス.


### クラス一覧(class list)

  * [FreeRegionList](#norB4teKMN)
  * [MasterFreeRegionList](#nosocWK0oe)
  * [SecondaryFreeRegionList](#noe_k17YTz)
  * [HumongousRegionSet](#noIKWQP6ym)
  * [MasterHumongousRegionSet](#no6ErWMEVi)


---
## <a name="norB4teKMN" id="norB4teKMN">FreeRegionList</a>

### 概要(Summary)
G1CollectedHeap クラス用の補助クラス.

空になった HeapRegion を回収する作業で使用されるクラス. HeapRegion のフリーリストを表す
(回収作業中は HeapRegion を FreeRegionList オブジェクト内に集め, 
 作業後に回収結果を SecondaryFreeRegionList や MasterFreeRegionList に追加する, という使われ方をする).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSets.hpp))
    //////////////////// FreeRegionList ////////////////////
    
    class FreeRegionList : public HeapRegionLinkedList {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている
(ただし, RegionResetter は StackObj クラスなので一時的なオブジェクト).

* 各 ConcurrentMark オブジェクトの _cleanup_list フィールド
* 各 RegionResetter オブジェクトの _local_free_list フィールド

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている
(ただし, ConcurrentMark::_cleanup_list フィールド以外は, 
StackObjクラスのフィールドもしくは局所変数としての生成なので一時的なオブジェクト).

* (ConcurrentMark クラスの _cleanup_list フィールドは, ポインタ型ではなく実体なので,
   ConcurrentMark オブジェクトの生成時に一緒に生成される)
* (RegionResetter クラスの _local_free_list フィールドは, ポインタ型ではなく実体なので,
   RegionResetter オブジェクトの生成時に一緒に生成される)
* G1ParNoteEndTask::work() 内 (局所変数として生成)
* ConcurrentMark::completeCleanup() 内 (局所変数として生成)
* G1CollectedHeap::free_collection_set() 内 (局所変数として生成)
* G1PrepareCompactClosure::free_humongous_region() 内 (局所変数として生成)




### 詳細(Details)
See: [here](../doxygen/classFreeRegionList.html) for details

---
## <a name="nosocWK0oe" id="nosocWK0oe">MasterFreeRegionList</a>

### 概要(Summary)
G1CollectedHeap クラス内で使用される補助クラス.

未使用の HeapRegion をつないでおくフリーリスト.
新しい HeapRegion が必要になるとここから確保が行われる.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSets.hpp))
    //////////////////// MasterFreeRegionList ////////////////////
    
    class MasterFreeRegionList : public FreeRegionList {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 G1CollectedHeap オブジェクトの _free_list フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // The master free list. It will satisfy all new region allocations.
      MasterFreeRegionList      _free_list;
```

#### 生成箇所(where its instances are created)
(G1CollectedHeap クラスの _free_list フィールドは, ポインタ型ではなく実体なので,
 G1CollectedHeap オブジェクトの生成時に一緒に生成される)

#### 使用箇所(where its instances are used)
* フリーリストから新しい HeapRegion を確保する処理
  
    * G1CollectedHeap::new_region()
    * G1CollectedHeap::new_region_try_secondary_free_list()

* 未使用の HeapRegion をフリーリストにつなぐ処理
  
    * G1CollectedHeap::expand()
    * G1CollectedHeap::release_gc_alloc_regions()
    * G1CollectedHeap::update_sets_after_freeing_regions()
    * G1CollectedHeap::append_secondary_free_list()

* 中身を再構築する処理 
  (ヒープの縮小時や Full GC 時に行われる. 
  tear_down_region_lists() でフリーリストを空にリセットし, rebuild_region_lists() に再構築する)

    * G1CollectedHeap::tear_down_region_lists()
    * G1CollectedHeap::rebuild_region_lists()




### 詳細(Details)
See: [here](../doxygen/classMasterFreeRegionList.html) for details

---
## <a name="noe_k17YTz" id="noe_k17YTz">SecondaryFreeRegionList</a>

### 概要(Summary)
G1CollectedHeap クラス内で使用される補助クラス.

Concurrent Marking の Cleanup 処理で回収された HeapRegion がつながれるフリーリスト (See: [here](no2935d4w.html) for details).
このリストの中身は適当なタイミングで MasterFreeRegionList に移動される.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSets.hpp))
    //////////////////// SecondaryFreeRegionList ////////////////////
    
    class SecondaryFreeRegionList : public FreeRegionList {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 G1CollectedHeap オブジェクトの _secondary_free_list フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // The secondary free list which contains regions that have been
      // freed up during the cleanup process. This will be appended to the
      // master free list when appropriate.
      SecondaryFreeRegionList   _secondary_free_list;
```

#### 生成箇所(where its instances are created)
(G1CollectedHeap クラスの _secondary_free_list フィールドは, ポインタ型ではなく実体なので,
 G1CollectedHeap オブジェクトの生成時に一緒に生成される)

#### 使用箇所(where its instances are used)
* 未使用の HeapRegion をフリーリストにつなぐ処理

    * G1CollectedHeap::secondary_free_list_add_as_tail()

* リストの中身を MasterFreeRegionList に移動する処理

    * G1CollectedHeap::append_secondary_free_list()




### 詳細(Details)
See: [here](../doxygen/classSecondaryFreeRegionList.html) for details

---
## <a name="noIKWQP6ym" id="noIKWQP6ym">HumongousRegionSet</a>

### 概要(Summary)
G1CollectedHeap クラス用の補助クラス.

GC 処理によって解放された Humongous 用の HeapRegion の情報を MasterHumongousRegionSet に反映させる処理で使用されるクラス
(GC 処理中は解放した HeapRegion の情報を HumongousRegionSet オブジェクト内に集めていき, 
 作業後に結果を MasterHumongousRegionSet に反映する, という使われ方をする).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSets.hpp))
    //////////////////// HumongousRegionSet ////////////////////
    
    class HumongousRegionSet : public HeapRegionSet {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている
(ただし, これらは全て StackObjクラスのフィールドもしくは局所変数としての生成なので一時的なオブジェクト).

* (G1PrepareCompactClosure クラスの _humongous_proxy_set フィールドは, ポインタ型ではなく実体なので,
  G1PrepareCompactClosure オブジェクトの生成時に一緒に生成される)

* G1ParNoteEndTask::work() 内 (局所変数として生成)




### 詳細(Details)
See: [here](../doxygen/classHumongousRegionSet.html) for details

---
## <a name="no6ErWMEVi" id="no6ErWMEVi">MasterHumongousRegionSet</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス(?? #TODO).

G1CollectedHeap クラス内で使用される補助クラス.

「Humongous な HeapRegion」の位置を覚えておくためのクラス
(Humongous オブジェクトの先頭に当たる HeapRegion を格納している).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSets.hpp))
    //////////////////// MasterHumongousRegionSet ////////////////////
    
    class MasterHumongousRegionSet : public HumongousRegionSet {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 G1CollectedHeap オブジェクトの _humongous_set フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // It keeps track of the humongous regions.
      MasterHumongousRegionSet  _humongous_set;
```

#### 生成箇所(where its instances are created)
(G1CollectedHeap クラスの _humongous_set フィールドは, ポインタ型ではなく実体なので,
 G1CollectedHeap オブジェクトの生成時に一緒に生成される)

#### 使用箇所(where its instances are used)
(G1CollectedHeap::verify_region_sets() 以外で中身を参照している箇所が見当たらないが...#TODO)

* Humangous HeapRegion の追加処理

  * G1CollectedHeap::humongous_obj_allocate_initialize_regions()

* 要素の削除処理 (remove_with_proxy() 及び update_from_proxy() によって行われる (See: HeapRegionSet))

  * G1CollectedHeap::free_humongous_region()
  * G1CollectedHeap::update_sets_after_freeing_regions()

* 中身をチェックする処理

  * G1CollectedHeap::verify_region_sets() 
    (及びそこから呼び出される VerifyRegionListsClosure::doHeapRegion())





### 詳細(Details)
See: [here](../doxygen/classMasterHumongousRegionSet.html) for details

---

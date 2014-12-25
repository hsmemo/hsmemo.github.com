---
layout: default
title: CollectionSetChooser クラス及びその補助クラス (CSetChooserCache, CollectionSetChooser)
---
[Top](../index.html)

#### CollectionSetChooser クラス及びその補助クラス (CSetChooserCache, CollectionSetChooser)

これらは, G1CollectedHeap 使用時における Garbage Collection 処理用の補助クラス.
より具体的に言うと, GC で処理対象となる HeapRegion ("Collection Set") を選択するクラス.


### クラス一覧(class list)

  * [CollectionSetChooser](#noTzDVmnfY)
  * [CSetChooserCache](#nodiMCjyzT)


---
## <a name="noTzDVmnfY" id="noTzDVmnfY">CollectionSetChooser</a>

### 概要(Summary)
G1CollectorPolicy クラス内で使用される補助クラス.

G1GC の Minor GC 時に, 処理対象となる HeapRegion を選択する (See: [here](no2935YzN.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/collectionSetChooser.hpp))
    class CollectionSetChooser: public CHeapObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
CollectionSetChooser::getNextMarkedRegion() を呼ぶと, 最も適切な HeapRegion を取得できる
(なお, このメソッドは G1CollectorPolicy_BestRegionsFirst::choose_collection_set() 内で(のみ)使用されている).

#### インスタンスの格納場所(where its instances are stored)
各 G1CollectorPolicy_BestRegionsFirst オブジェクトの _collectionSetChooser フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
G1CollectorPolicy_BestRegionsFirst::G1CollectorPolicy_BestRegionsFirst() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classCollectionSetChooser.html) for details

---
## <a name="nodiMCjyzT" id="nodiMCjyzT">CSetChooserCache</a>

### 概要(Summary)
CollectionSetChooser クラス内で使用されている補助クラス(ValueObjクラス).

候補の HeapRegion を優先度順に sort する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/collectionSetChooser.hpp))
    // We need to sort heap regions by collection desirability.
    
    class CSetChooserCache VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
要素を CSetChooserCache::insert() で追加した後,
CSetChooserCache::get_first() で最優先の候補を取得できる.
次の候補を得るには, CSetChooserCache::remove_first() を呼んでから
CSetChooserCache::get_first() を呼ぶ.

#### インスタンスの格納場所(where its instances are stored)
各 CollectionSetChooser オブジェクトの _cache フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(CollectionSetChooser オブジェクトの _cache フィールドは, ポインタ型ではなく実体なので,
 CollectionSetChooser オブジェクトの生成時に一緒に生成される)

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* CollectionSetChooser::getNextMarkedRegion()
* CollectionSetChooser::removeRegion()
* CollectionSetChooser::clearMarkedHeapRegions()
* CollectionSetChooser::fillCache()
* CollectionSetChooser::addRegionToCache()
* CollectionSetChooser::verify()

### 内部構造(Internal structure)
定義されているフィールドは以下のもののみ.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/collectionSetChooser.hpp))
      HeapRegion*  _cache[CacheLength];
      int          _occupancy; // number of region in cache
      int          _first; // "first" region in the cache
```




### 詳細(Details)
See: [here](../doxygen/classCSetChooserCache.html) for details

---

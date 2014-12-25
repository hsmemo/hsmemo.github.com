---
layout: default
title: OopMapCache クラス関連のクラス (OffsetClosure, InterpreterOopMap, OopMapCache, 及びそれらの補助クラス(OopMapCacheEntry, OopMapForCacheEntry, VerifyClosure, MaskFillerForNative))
---
[Top](../index.html)

#### OopMapCache クラス関連のクラス (OffsetClosure, InterpreterOopMap, OopMapCache, 及びそれらの補助クラス(OopMapCacheEntry, OopMapForCacheEntry, VerifyClosure, MaskFillerForNative))

これらは, Interpreter が GC 処理と協調するためのメタ情報(OopMap)を管理するためのクラス (See: [here](no7882AgC.html) and [here](no2114GzS.html) for details).

### 概要(Summary)
Interpreter による実行中には GC (及びその他の safepoint 処理) が発生する可能性があるため, 
「safepoint 停止した時点でどこにポインタ値が存在しているか」という情報(frame map または OopMap)がないといけない.

Interpreter の場合, OopMap 情報は GC 中に (OopMap が必要になった時点で) 生成する.
ただし, 毎回全て計算し直すのはもったいないので, 過去に計算したものをある程度メモイズして高速化している.

これらのクラスは以下の役割を担当する.

  * InterpreterOopMap
    
    OopMap 情報を表すクラス.
    どこが oop かを示す bitmap (bit mask) 情報を保持している.

  * OopMapCache
    
    過去に計算した InterpreterOopMap オブジェクトをキャッシュしているクラス.
    各 instanceKlass オブジェクトが1つずつ保持している.

    (method, bci) というペアをキーとして, キャッシュしている InterpreterOopMap オブジェクトを引くことができる.

  * OopMapCacheEntry
    
    OopMapCache を構成するエントリ.
    この中にキャッシュしている InterpreterOopMap オブジェクトの情報が格納されている.

実際に使う際には以下のようになる.

  1. まずキャッシュを引く (キャッシュにヒットしなければ, 新たに作成してキャッシュにロードする)

  2. キャッシュエントリからデータを (スレッドローカルな) InterpreterOopMap にコピーする

  3. コピーした InterpreterOopMap を使用する.
  
     (ほぼ read only なオブジェクトのように見えるので, キャッシュエントリそのものを使ってもいいような気もするが...
     
     キャッシュがあふれて evict する際にまだ使っているスレッドがいるか等を管理するのが面倒だから?? #TODO)

なお, InterpreterOopMap や OopMapCacheEntry 内での bitmap データの最適化として, 
bitmask の長さが 2words に収まる場合は InterpreterOopMap オブジェクト内のフィールドに格納している.
それを超える場合は以下のように heap 上に領域が取られる.

* OopMapCacheEntry の場合は, (GC をまたいで存在し続けるので) bit mask は C heap 上に確保される.
* OopMapCacheEntry ではない場合は, 
  (C heap 上での確保解放処理は遅いので) resource area 上に bit mask を確保する.
  (このため, InterpreterOopMap は一度の GC 中で作成され破棄されないといけない)

また, ENABBLE_ZAP_DEAD_LOCALS が #define されている場合は, 
bitmask は 2bit ずつ使われる (増えた 1bit は dead かどうかを示すために使われる)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/oopMapCache.hpp))
    // A Cache for storing (method, bci) -> oopMap.
    // The memory management system uses the cache when locating object
    // references in an interpreted frame.
    //
    // OopMapCache's are allocated lazily per instanceKlass.
    
    // The oopMap (InterpreterOopMap) is stored as a bit mask. If the
    // bit_mask can fit into two words it is stored in
    // the _bit_mask array, otherwise it is allocated on the heap.
    // For OopMapCacheEntry the bit_mask is allocated in the C heap
    // because these entries persist between garbage collections.
    // For InterpreterOopMap the bit_mask is allocated in
    // a resource area for better performance.  InterpreterOopMap
    // should only be created and deleted during same garbage collection.
    //
    // If ENABBLE_ZAP_DEAD_LOCALS is defined, two bits are used
    // per entry instead of one. In all cases,
    // the first bit is set to indicate oops as opposed to other
    // values. If the second bit is available,
    // it is set for dead values. We get the following encoding:
    //
    // 00 live value
    // 01 live oop
    // 10 dead value
    // 11 <unused>                                   (we cannot distinguish between dead oops or values with the current oop map generator)
```

### 備考(Notes)
現状では, キャッシュを介さずに OopMap を作成するというパスは用意されていない模様.

例えば, デバッグ用の関数として OopMapCache::compute_one_oop_map() というキャッシュを用いない関数も用意されているが, 
その中でもいったんキャッシュを介して計算しすぐにキャッシュを捨てるという実装になっている.


```cpp
    ((cite: hotspot/src/share/vm/interpreter/oopMapCache.cpp))
    void OopMapCache::compute_one_oop_map(methodHandle method, int bci, InterpreterOopMap* entry) {
      // Due to the invariants above it's tricky to allocate a temporary OopMapCacheEntry on the stack
      OopMapCacheEntry* tmp = NEW_C_HEAP_ARRAY(OopMapCacheEntry, 1);
      tmp->initialize();
      tmp->fill(method, bci);
      entry->resource_copy(tmp);
      FREE_C_HEAP_ARRAY(OopMapCacheEntry, tmp);
    }
```



### クラス一覧(class list)

  * [OffsetClosure](#noW30BuKzz)
  * [InterpreterOopMap](#nolEBxF8lm)
  * [OopMapCache](#noaFzE5fGF)
  * [OopMapCacheEntry](#noEf9HUdg1)
  * [OopMapForCacheEntry](#noQGHflSQA)
  * [VerifyClosure](#nof3223ZGa)
  * [MaskFillerForNative](#noiYvh2EIT)


---
## <a name="noW30BuKzz" id="noW30BuKzz">OffsetClosure</a>

### 概要(Summary)
InterpreterOopMap クラス用の補助クラス.

InterpreterOopMap 内の bitmap に対して iterate 処理を行うための Closure クラス (の基底クラス).

なお, このクラス自体は abstract class であり, 実際に使われるのは以下のサブクラス.

  * InterpreterFrameClosure	: GC 時に interpreter frame 内の oop を調べるクラス
  * VerifyClosure : デバッグ用に oopmap の verify 処理を行うクラス


```cpp
    ((cite: hotspot/src/share/vm/interpreter/oopMapCache.hpp))
    class OffsetClosure  {
```

### 使われ方(Usage)
InterpreterOopMap::iterate_oop() および InterpreterOopMap::iterate_all() が, 引数として OffsetClosure 型の値を受け取る.


```cpp
    ((cite: hotspot/src/share/vm/interpreter/oopMapCache.hpp))
      void iterate_oop(OffsetClosure* oop_closure);
    ...
    #ifdef ENABLE_ZAP_DEAD_LOCALS
      void iterate_all(OffsetClosure* oop_closure, OffsetClosure* value_closure, OffsetClosure* dead_closure);
    #endif
```




### 詳細(Details)
See: [here](../doxygen/classOffsetClosure.html) for details

---
## <a name="nolEBxF8lm" id="nolEBxF8lm">InterpreterOopMap</a>

### 概要(Summary)
GC 処理中に使用される一時オブジェクト(ResourceObjクラス).

Interpreter が実行しているコード地点における OopMap 情報を表す.


```cpp
    ((cite: hotspot/src/share/vm/interpreter/oopMapCache.hpp))
    class InterpreterOopMap: ResourceObj {
```

### 内部構造(Internal structure)
どこが oop かを示す bitmap (bit mask) 情報を保持している.

内部には 2 words の大きさの配列を保持しており, ここにその bitmap 情報が格納される.

(より正確に言うと, 
bitmask の大きさが 2 words 以下であれば, この配列内に直接 bitmask を格納する.
それより大きい場合は, C heap 上に取った領域へのポインタをここに格納する)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/oopMapCache.hpp))
        N                = 2,                // the number of words reserved
                                             // for inlined mask storage
```


```cpp
    ((cite: hotspot/src/share/vm/interpreter/oopMapCache.hpp))
      intptr_t       _bit_mask[N];    // the bit mask if
                                      // mask_size <= small_mask_limit,
                                      // ptr to bit mask otherwise
                                      // "protected" so that sub classes can
                                      // access it without using trickery in
                                      // methd bit_mask().
```




### 詳細(Details)
See: [here](../doxygen/classInterpreterOopMap.html) for details

---
## <a name="noaFzE5fGF" id="noaFzE5fGF">OopMapCache</a>

### 概要(Summary)
過去に計算した InterpreterOopMap オブジェクトをキャッシュしておくためのクラス.

(method, bci) というペアをキーとして, キャッシュしている InterpreterOopMap オブジェクトを引くことができる.


```cpp
    ((cite: hotspot/src/share/vm/interpreter/oopMapCache.hpp))
    class OopMapCache : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 instanceKlass オブジェクトの _oop_map_cache フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
      OopMapCache*    volatile _oop_map_cache;   // OopMapCache for all methods in the klass (allocated lazily)
```

### 内部構造(Internal structure)
(内部的には closed hash になっている模様 #TODO)

(なお, 現状ではハッシュのバケット数は 32. また collision 時の re-hash は 3回まで)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/oopMapCache.hpp))
      enum { _size        = 32,     // Use fixed size for now
             _probe_depth = 3       // probe depth in case of collisions
      };
```




### 詳細(Details)
See: [here](../doxygen/classOopMapCache.html) for details

---
## <a name="noEf9HUdg1" id="noEf9HUdg1">OopMapCacheEntry</a>

### 概要(Summary)
OopMapCache クラス内で使用される補助クラス.

OopMapCache オブジェクト内に格納されるハッシュテーブル・エントリ.

この OopMapCacheEntry 内に, 実際のキャッシュしている InterpreterOopMap オブジェクトの情報が格納される.
1つの OopMapCacheEntry オブジェクトが 1つの InterpreterOopMap オブジェクトに対応する.


```cpp
    ((cite: hotspot/src/share/vm/interpreter/oopMapCache.cpp))
    class OopMapCacheEntry: private InterpreterOopMap {
```

### 内部構造(Internal structure)
OopMapCacheEntry に必要なデータ構造は InterpreterOopMap とほぼ同じなので, 
InterpreterOopMap を private 継承して実装されている.




### 詳細(Details)
See: [here](../doxygen/classOopMapCacheEntry.html) for details

---
## <a name="noQGHflSQA" id="noQGHflSQA">OopMapForCacheEntry</a>

### 概要(Summary)
OopMapCacheEntry クラス用の補助クラス.

OopMapCacheEntry を作成する処理で使用されるクラス.
実際に OopMap を計算する処理を担当する.

(なお, こちらは非 native メソッド用. MaskFillerForNative も参照)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/oopMapCache.cpp))
    // Implementation of OopMapForCacheEntry
    // (subclass of GenerateOopMap, initializes an OopMapCacheEntry for a given method and bci)
    
    class OopMapForCacheEntry: public GenerateOopMap {
```

### 使われ方(Usage)
OopMapCacheEntry::fill() 内で(のみ)使用されている.

### 内部構造(Internal structure)
実際の処理はスーパークラスである GenerateOopMap に丸投げしている.




### 詳細(Details)
See: [here](../doxygen/classOopMapForCacheEntry.html) for details

---
## <a name="nof3223ZGa" id="nof3223ZGa">VerifyClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

OopMapCacheEntry の中身の verify 処理を行う Closure.


```cpp
    ((cite: hotspot/src/share/vm/interpreter/oopMapCache.cpp))
    // Implementation of InterpreterOopMap and OopMapCacheEntry
    
    class VerifyClosure : public OffsetClosure {
```

### 使われ方(Usage)
OopMapCacheEntry::verify_mask() 内で(のみ)使用されている.

(なお, この関数自体も assert() 内でしか呼び出されない)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/oopMapCache.cpp))
      // verify bit mask
      assert(verify_mask(vars, stack, max_locals, stack_top), "mask could not be verified");
```




### 詳細(Details)
See: [here](../doxygen/classVerifyClosure.html) for details

---
## <a name="noiYvh2EIT" id="noiYvh2EIT">MaskFillerForNative</a>

### 概要(Summary)
OopMapCacheEntry クラス用の補助クラス.

OopMapCacheEntry を作成する処理で使用されるクラス.
実際に OopMap を計算する処理を担当する.

(なお, こちらは native メソッド用. OopMapForCacheEntry も参照)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/oopMapCache.cpp))
    class MaskFillerForNative: public NativeSignatureIterator {
```

### 使われ方(Usage)
OopMapCacheEntry::fill_for_native() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classMaskFillerForNative.html) for details

---

---
layout: default
title: WaterMark クラス 
---
[Top](../index.html)

#### WaterMark クラス 



---
## <a name="noBN55tyaw" id="noBN55tyaw">WaterMark</a>

### 概要(Summary)
保守運用機能のためのクラス (AllocationProfiler クラス用の補助クラス). (See: AllocationProfiler)

直近の GC が完了した時点でのヒープの終端(top 位置)を覚えておくためのクラス
(この情報は, 直近の GC 以降に増えたオブジェクトだけを対象に何らかの処理を行いたい, という際に使われる).


```cpp
    ((cite: hotspot/src/share/vm/memory/watermark.hpp))
    // A water mark points into a space and is used during GC to keep track of
    // progress.
    ...
    class WaterMark VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
OneContigSpaceCardGeneration オブジェクトの _last_gc フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/generation.hpp))
    class OneContigSpaceCardGeneration: public CardGeneration {
    ...
      WaterMark  _last_gc;                // watermark between objects allocated before
                                          // and after last GC.
```

#### 使用箇所(where its instances are used)
GC 終了時に, OneContigSpaceCardGeneration::gc_epilogue() 内でその時点での top 位置が記録される.


```cpp
    ((cite: hotspot/src/share/vm/memory/generation.cpp))
    void OneContigSpaceCardGeneration::gc_epilogue(bool full) {
      _last_gc = WaterMark(the_space(), the_space()->top());
```

そして, この情報は AllocationProfiler::iterate_since_last_gc() 内で(のみ)使用されている
(なお, この関数ではクラス毎のインスタンス生成量情報をカウントアップするために用いられている).

#### 参考(for your information): AllocationProfiler::iterate_since_last_gc()
See: [here](no3269KhL.html) for details
#### 参考(for your information): GenCollectedHeap::object_iterate_since_last_GC()
See: [here](no3269XrR.html) for details
#### 参考(for your information): DefNewGeneration::object_iterate_since_last_GC()
See: [here](no3269k1X.html) for details
#### 参考(for your information): OneContigSpaceCardGeneration::object_iterate_since_last_GC()
See: [here](no3269x_d.html) for details
#### 参考(for your information): ContiguousSpace::object_iterate_from()
See: [here](no3269-Jk.html) for details
### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を含む.
(そして, メソッドはこれらのフィールドへのアクセサメソッドのみ)


```cpp
    ((cite: hotspot/src/share/vm/memory/watermark.hpp))
      HeapWord* _point;
      Space*    _space;
```





### 詳細(Details)
See: [here](../doxygen/classWaterMark.html) for details

---

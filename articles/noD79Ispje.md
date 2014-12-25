---
layout: default
title: SpaceCounters 及びその補助クラスの定義 (SpaceCounters, MutableSpaceUsedHelper)
---
[Top](../index.html)

#### SpaceCounters 及びその補助クラスの定義 (SpaceCounters, MutableSpaceUsedHelper)

これらは, 保守運用機能のためのクラス.
より具体的に言うと, MutableSpace オブジェクトに関する情報を PerfData 形式で出力するためのクラス
(See: [here](no3718kvd.html) for details) (See: PerfData).


### クラス一覧(class list)

  * [SpaceCounters](#nofHgfzXhk)
  * [MutableSpaceUsedHelper](#noO5iL1Ttm)


---
## <a name="nofHgfzXhk" id="nofHgfzXhk">SpaceCounters</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData 管理用のクラス).

MutableSpace に関する PerfData を格納しておくためのクラス.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/spaceCounters.hpp))
    // A SpaceCounter is a holder class for performance counters
    // that track a space;
    
    class SpaceCounters: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
MutableSpace (やそのサブクラス) のオブジェクトを保持しているクラス内に一緒に保持されている.

* PSYoungGen

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psYoungGen.hpp))
    class PSYoungGen : public CHeapObj {
    ...
      SpaceCounters*            _eden_counters;
      SpaceCounters*            _from_counters;
      SpaceCounters*            _to_counters;
```

* PSOldGen

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psOldGen.hpp))
    class PSOldGen : public CHeapObj {
    ...
      SpaceCounters*           _space_counters;
```

### 内部構造(Internal structure)
内部には, 記録対象の MutableSpace と, そのパフォーマンスカウンタとして使う PerfVariable 2 個を保持している.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/spaceCounters.hpp))
      PerfVariable*      _capacity;
      PerfVariable*      _used;
```

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/spaceCounters.hpp))
      MutableSpace*     _object_space;
```

これらの PerfVariable には, (そのフィールド名の通り) 対応する MutableSpace の最大量(capacity)と現在使用量(used)が記録される.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/spaceCounters.hpp))
      inline void update_capacity() {
        _capacity->set_value(_object_space->capacity_in_bytes());
      }
    
      inline void update_used() {
        _used->set_value(_object_space->used_in_bytes());
      }
```

正確には, 可変でない項目も入れると, 各 MutableSpace の以下の情報が格納されている.
(現在の領域長や現在の使用量は実行時に変わるので GC 後などに変更している. それ以外は生成時に指定した値で固定).

* 領域名("name")
* 最大領域長("maxCapacity")
* 現在の領域長("capacity")
* 現在の使用量("used")
* 初期領域長("initCapacity") 

それぞれ以下の名前でアクセス可能 
(${n} や ${m} の箇所には 0, 1, ... といった数字が入る. 例えば sun.gc.generation.0.space.1.name 等)

  * sun.gc.generation.${n}.space.${m}.name
  * sun.gc.generation.${n}.space.${m}.maxCapacity
  * sun.gc.generation.${n}.space.${m}.capacity
  * sun.gc.generation.${n}.space.${m}.used
  * sun.gc.generation.${n}.space.${m}.initCapacity

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/spaceCounters.cpp))
        const char* cns = PerfDataManager::name_space(gc->name_space(), "space",
                                                      ordinal);
    
        _name_space = NEW_C_HEAP_ARRAY(char, strlen(cns)+1);
        strcpy(_name_space, cns);
    
        const char* cname = PerfDataManager::counter_name(_name_space, "name");
        PerfDataManager::create_string_constant(SUN_GC, cname, name, CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "maxCapacity");
        PerfDataManager::create_constant(SUN_GC, cname, PerfData::U_Bytes,
                                         (jlong)max_size, CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "capacity");
        _capacity = PerfDataManager::create_variable(SUN_GC, cname,
                                       PerfData::U_Bytes,
                                       _object_space->capacity_in_bytes(), CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "used");
        _used = PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes,
                                        new MutableSpaceUsedHelper(_object_space),
                                        CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "initCapacity");
        PerfDataManager::create_constant(SUN_GC, cname, PerfData::U_Bytes,
                                         _object_space->capacity_in_bytes(), CHECK);
```




### 詳細(Details)
See: [here](../doxygen/classSpaceCounters.html) for details

---
## <a name="noO5iL1Ttm" id="noO5iL1Ttm">MutableSpaceUsedHelper</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData 管理用のクラス).

MutableSpace を PerfLongSampleHelper (のサブクラス) として使うためのラッパークラス.
(より具体的に言うと, MutableSpace の使用量情報(used)を PerfVariable で記録するためのクラス).

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/spaceCounters.hpp))
    class MutableSpaceUsedHelper: public PerfLongSampleHelper {
```

### 内部構造(Internal structure)
やってることは, PerfLongSampleHelper::take_sample() を MutableSpace::used_in_bytes() に変換するだけ.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/spaceCounters.hpp))
        inline jlong take_sample() {
          return _m->used_in_bytes();
        }
```




### 詳細(Details)
See: [here](../doxygen/classMutableSpaceUsedHelper.html) for details

---

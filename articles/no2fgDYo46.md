---
layout: default
title: GSpaceCounters 及びその補助クラス (GSpaceCounters, GenerationUsedHelper)
---
[Top](../index.html)

#### GSpaceCounters 及びその補助クラス (GSpaceCounters, GenerationUsedHelper)

これらは, 保守運用機能のためのクラス.
より具体的に言うと, Generation オブジェクトに関する情報を PerfData 形式で出力するためのクラス
(See: [here](no3718kvd.html) for details) (See: PerfData).


### クラス一覧(class list)

  * [GSpaceCounters](#noeXHrIFoW)
  * [GenerationUsedHelper](#noWlGfehZ8)


---
## <a name="noeXHrIFoW" id="noeXHrIFoW">GSpaceCounters</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData 管理用のクラス).

Generation と1対1対応するような Space に関する PerfData を格納しておくためのクラス
(実際の使われ方としては, CompactibleFreeListSpace に関する PerfData を格納しておくためのクラス).

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gSpaceCounters.hpp))
    // A GSpaceCounter is a holder class for performance counters
    // that track a space;
    
    class GSpaceCounters: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 ConcurrentMarkSweepGeneration オブジェクトの _space_counters フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/concurrentMarkSweep/concurrentMarkSweepGeneration.hpp))
    class ConcurrentMarkSweepGeneration: public CardGeneration {
    ...
      GSpaceCounters*          _space_counters;
```

#### 生成箇所(where its instances are created)
以下の箇所で生成されている.

* ConcurrentMarkSweepGeneration::initialize_performance_counters()

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/concurrentMarkSweep/concurrentMarkSweepGeneration.cpp))
    void ConcurrentMarkSweepGeneration::initialize_performance_counters() {
    ...
      _space_counters = new GSpaceCounters(gen_name, 0,
                                           _virtual_space.reserved_size(),
                                           this, _gen_counters);
```

* CMSPermGenGen::initialize_performance_counters()

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/concurrentMarkSweep/cmsPermGen.cpp))
    void CMSPermGenGen::initialize_performance_counters() {
    ...
      _space_counters = new GSpaceCounters(gen_name, 0,
                                           _virtual_space.reserved_size(),
                                           this, _gen_counters);
```

### 内部構造(Internal structure)
内部には, 記録対象の Generation と, そのパフォーマンスカウンタとして使う PerfVariable 2 個を保持している.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gSpaceCounters.hpp))
      PerfVariable*      _capacity;
      PerfVariable*      _used;
```

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gSpaceCounters.hpp))
      Generation*       _gen;
```

これらの PerfVariable には, (そのフィールド名の通り) 対応する Generation の最大量(capacity)と現在使用量(used)が記録される.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gSpaceCounters.hpp))
      inline void update_capacity() {
        _capacity->set_value(_gen->capacity());
      }
    
      inline void update_used() {
        _used->set_value(_gen->used());
      }
```

正確には, 可変でない項目も入れると, 各 Space の以下の情報が格納されている
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
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gSpaceCounters.cpp))
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
                                                     _gen->capacity(), CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "used");
        if (sampled) {
          _used = PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes,
                                                   new GenerationUsedHelper(_gen),
                                                   CHECK);
        }
        else {
          _used = PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes,
                                                   (jlong)0, CHECK);
        }
    
        cname = PerfDataManager::counter_name(_name_space, "initCapacity");
        PerfDataManager::create_constant(SUN_GC, cname, PerfData::U_Bytes,
                                         _gen->capacity(), CHECK);
```



### 詳細(Details)
See: [here](../doxygen/classGSpaceCounters.html) for details

---
## <a name="noWlGfehZ8" id="noWlGfehZ8">GenerationUsedHelper</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData 管理用のクラス).

Generation を PerfLongSampleHelper (のサブクラス) として使うためのラッパークラス
(より具体的に言うと, Generation の使用量情報(used)を PerfVariable で記録するためのクラス).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gSpaceCounters.hpp))
    class GenerationUsedHelper : public PerfLongSampleHelper {
```

### 内部構造(Internal structure)
やってることは, PerfLongSampleHelper::take_sample() を Generation::used() に変換するだけ.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gSpaceCounters.hpp))
        inline jlong take_sample() {
          return _gen->used();
        }
```




### 詳細(Details)
See: [here](../doxygen/classGenerationUsedHelper.html) for details

---

---
layout: default
title: CSpaceCounters 及びその補助クラス (CSpaceCounters, ContiguousSpaceUsedHelper)
---
[Top](../index.html)

#### CSpaceCounters 及びその補助クラス (CSpaceCounters, ContiguousSpaceUsedHelper)

これらは, 保守運用機能のためのクラス.
より具体的に言うと, ContiguousSpace オブジェクトに関する情報を PerfData 形式で出力するためのクラス
(See: [here](no3718kvd.html) for details) (See: PerfData).


### クラス一覧(class list)

  * [CSpaceCounters](#no5JlkW4nK)
  * [ContiguousSpaceUsedHelper](#nojdGa_E0o)


---
## <a name="no5JlkW4nK" id="no5JlkW4nK">CSpaceCounters</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData 管理用のクラス).

ContiguousSpace に関する PerfData を格納しておくためのクラス.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/cSpaceCounters.hpp))
    // A CSpaceCounters is a holder class for performance counters
    // that track a space;
    
    class CSpaceCounters: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ContiguousSpace (やそのサブクラス) のオブジェクトを保持しているクラス内に一緒に保持されている.

* DefNewGeneration

```cpp
    ((cite: hotspot/src/share/vm/memory/defNewGeneration.hpp))
    class DefNewGeneration: public Generation {
    ...
      CSpaceCounters*      _eden_counters;
      CSpaceCounters*      _from_counters;
      CSpaceCounters*      _to_counters;
```

* TenuredGeneration

```cpp
    ((cite: hotspot/src/share/vm/memory/tenuredGeneration.hpp))
    class TenuredGeneration: public OneContigSpaceCardGeneration {
    ...
      CSpaceCounters*       _space_counters;
```

* CompactingPermGenGen

```cpp
    ((cite: hotspot/src/share/vm/memory/compactingPermGenGen.hpp))
    class CompactingPermGenGen: public OneContigSpaceCardGeneration {
    ...
      CSpaceCounters*      _space_counters;
```


### 内部構造(Internal structure)
内部には, 記録対象の ContigousSpace と, そのパフォーマンスカウンタとして使う PerfVariable 2 個を保持している.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/cSpaceCounters.hpp))
      PerfVariable*      _capacity;
      PerfVariable*      _used;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/cSpaceCounters.hpp))
      ContiguousSpace*     _space;
```

これらの PerfVariable には, (そのフィールド名の通り) 対応する ContiguousSpace の最大量(capacity)と現在使用量(used)が記録される.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/cSpaceCounters.hpp))
      inline void update_capacity() {
        _capacity->set_value(_space->capacity());
      }
    
      inline void update_used() {
        _used->set_value(_space->used());
      }
```

正確には, 可変でない項目も入れると, 各 ContiguousSpace の以下の情報が格納されている.
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
    ((cite: hotspot/src/share/vm/gc_implementation/shared/cSpaceCounters.cpp))
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
                                                     _space->capacity(), CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "used");
        _used = PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes,
                                        new ContiguousSpaceUsedHelper(_space),
                                        CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "initCapacity");
        PerfDataManager::create_constant(SUN_GC, cname, PerfData::U_Bytes,
                                         _space->capacity(), CHECK);
```




### 詳細(Details)
See: [here](../doxygen/classCSpaceCounters.html) for details

---
## <a name="nojdGa_E0o" id="nojdGa_E0o">ContiguousSpaceUsedHelper</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData 管理用のクラス).

ContiguousSpace を PerfLongSampleHelper (のサブクラス) として使うためのラッパークラス
(より具体的に言うと, ContiguousSpace の使用量情報(used)を PerfVariable で記録するためのクラス).

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/cSpaceCounters.hpp))
    class ContiguousSpaceUsedHelper : public PerfLongSampleHelper {
```

### 内部構造(Internal structure)
やってることは, PerfLongSampleHelper::take_sample() を ContiguousSpace::used() に変換するだけ.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/cSpaceCounters.hpp))
        inline jlong take_sample() {
          return _space->used();
        }
```




### 詳細(Details)
See: [here](../doxygen/classContiguousSpaceUsedHelper.html) for details

---

---
layout: default
title: ParCompactionManager クラス 
---
[Top](../index.html)

#### ParCompactionManager クラス 



---
## <a name="nomadWRBeb" id="nomadWRBeb">ParCompactionManager</a>

### 概要(Summary)
ParallelScavengeHeap の Parallel Compaction 処理
(UseParallelOldGC オプションが指定されている場合の Major GC 処理)で使用される補助クラス(See: [here](no28916egj.html) and [here](no28916Gft.html) for details).

Mark 処理や Compact 処理を並列化するための機能を提供している.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psCompactionManager.hpp))
    class ParCompactionManager : public CHeapObj {
```

### 内部構造(Internal structure)
内部には OverflowTaskQueue<oop> 型のフィールド (_marking_stack) を持っており, 
mark 処理で見つかったポインタはこのスタックに積まれる.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psCompactionManager.hpp))
      OverflowTaskQueue<oop>        _marking_stack;
```

また, 内部には ObjArrayTaskQueue 型のフィールド (_objarray_stack) も持っており, 
長すぎるために一度に処理したくないポインタ配列については
(いったん処理を pending した状態で) このスタックに積まれる.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psCompactionManager.hpp))
      ObjArrayTaskQueue             _objarray_stack;
```




### 詳細(Details)
See: [here](../doxygen/classParCompactionManager.html) for details

---

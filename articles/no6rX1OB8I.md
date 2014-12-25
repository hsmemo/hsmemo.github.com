---
layout: default
title: IsGCActiveMark クラス 
---
[Top](../index.html)

#### IsGCActiveMark クラス 



---
## <a name="noxggx3Vnx" id="noxggx3Vnx">IsGCActiveMark</a>

### 概要(Summary)
Garbage Collection 処理用の補助クラス(StackObjクラス).

ソースコード中のあるスコープの間だけ, 
CollectedHeap オブジェクトの _is_gc_active フィールドを true にするためのクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/isGCActiveMark.hpp))
    // This class provides a method for block structured setting of the
    // _is_gc_active state without requiring accessors in CollectedHeap
    
    class IsGCActiveMark : public StackObj {
```

なお, is_gc_active は現在 STW 型の GC を実行中かどうか, を示すフィールド (GC 実行中には true にする).

```cpp
    ((cite: hotspot/src/share/vm/gc_interface/collectedHeap.hpp))
      // Returns "true" iff there is a stop-world GC in progress.  (I assume
      // that it should answer "false" for the concurrent part of a concurrent
      // collector -- dld).
      bool is_gc_active() const { return _is_gc_active; }
```

### 内部構造(Internal structure)
Universe::heap() に設定されている CollectedHeap オブジェクトに対し, 
コンストラクタで CollectedHeap::_is_gc_active を true に設定し, デストラクタで false に戻している.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/isGCActiveMark.hpp))
      IsGCActiveMark() {
        CollectedHeap* heap = Universe::heap();
        assert(!heap->is_gc_active(), "Not reentrant");
        heap->_is_gc_active = true;
      }
    
      ~IsGCActiveMark() {
        CollectedHeap* heap = Universe::heap();
        assert(heap->is_gc_active(), "Sanity");
        heap->_is_gc_active = false;
      }
```




### 詳細(Details)
See: [here](../doxygen/classIsGCActiveMark.html) for details

---

---
layout: default
title: Prefetch クラス 
---
[Top](../index.html)

#### Prefetch クラス 



---
## <a name="noIlcCfRcK" id="noIlcCfRcK">Prefetch</a>

### 概要(Summary)
メモリのプリフェッチ処理用のユーティリティ・クラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

プリフェッチ処理を実行する関数を提供している.


```
    ((cite: hotspot/src/share/vm/runtime/prefetch.hpp))
    // If calls to prefetch methods are in a loop, the loop should be cloned
    // such that if Prefetch{Scan,Copy}Interval and/or PrefetchFieldInterval
    // say not to do prefetching, these methods aren't called.  At the very
    // least, they take up a memory issue slot.  They should be implemented
    // as inline assembly code: doing an actual call isn't worth the cost.
    
    class Prefetch : AllStatic {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
以下の2つのメソッドを提供している.
ただし具体的な処理は os や cpu に依存するため, これらは os_cpu/ 以下で定義されている.


```
    ((cite: hotspot/src/share/vm/runtime/prefetch.hpp))
      // Prefetch anticipating read; must not fault, semantically a no-op
      static void read(void* loc, intx interval);
    
      // Prefetch anticipating write; must not fault, semantically a no-op
      static void write(void* loc, intx interval);
```




### 詳細(Details)
See: [here](../doxygen/classPrefetch.html) for details

---

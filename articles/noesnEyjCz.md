---
layout: default
title: AllocationStats クラス 
---
[Top](../index.html)

#### AllocationStats クラス 



---
## <a name="nolrqevRhy" id="nolrqevRhy">AllocationStats</a>

### 概要(Summary)
メモリ確保に関する統計情報を格納しておくためのクラス(?).

(なお現状では CMS 専用のクラスである模様)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/allocationStats.hpp))
    class AllocationStats VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 FreeList オブジェクトの _allocation_stats フィールドに(のみ)格納されている. (<= これは CMS 用のクラス).




### 詳細(Details)
See: [here](../doxygen/classAllocationStats.html) for details

---

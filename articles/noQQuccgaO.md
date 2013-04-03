---
layout: default
title: MemoryUsage クラス 
---
[Top](../index.html)

#### MemoryUsage クラス 



---
## <a name="nogA-ocIhF" id="nogA-ocIhF">MemoryUsage</a>

### 概要(Summary)
Platform MXBean 機能のためのクラス.
より具体的に言うと, ParallelScavengeHeap 使用時の java.lang.management.MemoryUsage クラスの実装を担当するクラス
(つまり, メモリ空間の使用状況を表すためのクラス).
(See: [here](no2114twV.html) for details)
(See: [here](no211477i.html) for details)


```
    ((cite: hotspot/src/share/vm/services/memoryUsage.hpp))
    class MemoryUsage VALUE_OBJ_CLASS_SPEC {
```

(なお, Platform MXBean としての機能以外に,
 HotSpot の内部的にメモリ使用量等を記録しておきたい場面で
 MemoryUsage オブジェクトとして管理しているケースもある模様)

### 内部構造(Internal structure)
内部には4つのフィールド(のみ)を保持する
(これは java.lang.management.MemoryUsage の 4つの値に直接対応).

  * initSize  -- 初期サイズ [byte]
  * used      -- 現在の使用量 [byte]
  * committed -- 現在のヒープサイズ [byte]
  * maxSize   -- 最大拡張時のヒープサイズ [byte]

```
    ((cite: hotspot/src/share/vm/services/memoryUsage.hpp))
    // A memory usage contains the following attributes about memory usage:
    //  initSize - represents the initial amount of memory (in bytes) that
    //     the Java virtual machine requests from the operating system
    //     for memory management.  The Java virtual machine may request
    //     additional memory from the operating system later when appropriate.
    //     Its value may be undefined.
    //  used      - represents the amount of memory currently used (in bytes).
    //  committed - represents the amount of memory (in bytes) that is
    //     guaranteed to be available for use by the Java virtual machine.
    //     The amount of committed memory may change over time (increase
    //     or decrease).  It is guaranteed to be greater than or equal
    //     to initSize.
    //  maxSize   - represents the maximum amount of memory (in bytes)
    //     that can be used for memory management. The maximum amount of
    //     memory for memory management could be less than the amount of
    //     committed memory.  Its value may be undefined.
```


```
    ((cite: hotspot/src/share/vm/services/memoryUsage.hpp))
      size_t _initSize;
      size_t _used;
      size_t _committed;
      size_t _maxSize;
```




### 詳細(Details)
See: [here](../doxygen/classMemoryUsage.html) for details

---
